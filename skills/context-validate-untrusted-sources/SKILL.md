---
name: context-validate-untrusted-sources
description: >-
  Requires Zod at the Matreshka BFF boundary for data from untrusted sources:
  client context-values and external API or service payloads. Context schema
  lists keys the client may write (inputs, context.local / ref.local) and
  checks their types. Server-only keys stay out of that schema; validate
  external responses with a separate Zod schema when the client must not write
  those fields. Trusted server writes rely on TypeScript. Use when adding
  Context, inputs, local context updates, third-party data, or reviewing
  incoming values.
---

# Zod для данных из недоверенных источников (Matreshka BFF)

## Правило

Данные из недоверенного источника проверяй Zod во время выполнения, до использования в приложении. TypeScript проверяет только код, который пишет сам сервер.

Недоверенный источник — не место, где выполняется код, а то, можно ли доверять данным. К таким источникам относятся клиент, сторонний API, внешний сервис и любой другой ресурс, чей ответ может не совпасть с ожидаемым типом. Валидация ловит намеренно некорректные сообщения, неожиданный формат, смену API и ошибки интеграции.

Внутреннюю запись сервера (`data()`, `context.set`, `ref.set`, `replace`) схема не проверяет: тип такой записи задаёт TypeScript.

## Клиент и `schema` Context

Клиент — один из недоверенных источников. Поле `schema` у `Context` для него делает две вещи: перечисляет ключи, которые клиенту разрешено менять, и проверяет тип значения. Входящее `context-values` обновляет только ключи из этой схемы.

Ключ обязан быть в `schema`, если клиент его меняет:

- компонент пишет в контекст. Сейчас это input: `textInput`, `passwordInput`, `numberInput`, `textarea`, `textEditor` через `ref`;
- ключ записан локальным действием `context.local.set` / `toggle` / `inc` / `dec` или `ref.local.set` / `toggle` / `inc` / `dec`.

Локальное действие сразу обновляет зеркало контекста на клиенте и затем синхронизирует его с BFF тем же `context-values`.

Если один ключ меняют и клиент, и сервер, он всё равно входит в `schema`. Наличие ключа в схеме серверную запись не ограничивает.

Ключ, который пишет только BFF, в `schema` не включай. Чтение ref (`text`, `when.equals` и другие conditions) клиентскую запись не создаёт. Контекст без клиентских записей оставляй без `schema`.

### Почему список ключей явный

Сериализация любой ссылки на контекст привязывает клиента ко всему этому контексту: компонент, condition, плейсхолдер `@{…}`, payload локального действия. После привязки клиент получает полный snapshot (`context-init`), а не отдельный ключ.

Дальше он может прислать `context-values` по любому ключу: значение другого типа или ключ, которого в данных нет. Типы `textInput(ref)` и `context.local.*` ограничивают только исходный код BFF. Сообщение с клиента они не проверяют.

Чтение тоже открывает весь контекст. Ключи только для BFF в схему не добавляй: иначе клиент сможет записать в них значение подходящего типа.

`schema` не скрывает данные. Ключ вне схемы всё равно попадает в snapshot, если он есть в данных контекста. То, что клиент видеть не должен, в сериализуемый контекст не клади.

Без `schema` сервер не отсекает ключи входящего `context-values`. Отсутствие схемы — не запрет записи.

## Сторонний API и другие внешние ответы

Ответ внешнего сервиса остаётся недоверенным, даже если запрос выполняет сервер и у клиента интеграции есть TypeScript-типы. Перед записью в Context разбери payload как `unknown` и проверь Zod. `parse()` бросает ошибку на границе получения данных, а не позже в компоненте.

Ключи `schema` у Context одновременно разрешены для клиентской записи. Поля ответа, которые клиент менять не должен, в эту схему не добавляй: проверь их отдельной Zod-схемой и запиши результат через `set` или `replace`.

Если клиенту эти поля менять можно, ту же схему можно повесить на Context и вызвать `context.schema.parse`.

## Как описывать схему клиентских ключей

Тип Zod совпадает с тем, что контрол или local action реально отправляет:

- строковый input — `z.string()` (пустая строка допустима);
- `numberInput`, `inc`, `dec` — `z.number()`;
- `toggle` — `z.boolean()`;
- `local.set` объекта или массива — та же форма, что в patch.

Вложенный путь (`user.email`, `items[0].qty`) опиши деревом `z.object` / `z.array`, чтобы путь существовал в схеме.

Схема проверяет тип и право клиента писать в ключ. Формат вроде email на каждый символ и бизнес-правила остаются на submit в server handler.

## Примеры

Клиентские ключи и серверные поля в одном контексте:

```typescript
context = new Context({
  schema: z.object({
    query: z.string(),
    quantity: z.number(),
    enabled: z.boolean(),
  }),
  data: async () => ({
    query: "",
    quantity: 1,
    enabled: false,
    orderId: "srv-1",
    role: "viewer",
  }),
});

textInput({ ref: this.context.ref("query") });

button({ onClick: this.context.local.inc("quantity") }, [text("+")]);

button({ onClick: this.context.ref("enabled").local.toggle() }, [
  text("Вкл"),
]);

// orderId и role пишет только BFF
this.context.set({ role: "editor" });
```

Контекст только с серверными записями:

```typescript
context = new Context({
  data: async () => ({ title: "Счёт", balance: 0 }),
});

text(this.context.ref("title"));
```

Ответ внешнего API с полями, которые клиент менять не должен:

```typescript
const deliverySchema = z.object({
  status: z.enum(["pending", "shipped", "delivered"]),
  trackingNumber: z.string(),
});

context = new Context({
  data: async () => ({
    status: "pending",
    trackingNumber: "",
  }),
});

const response = await fetch(deliveryApiUrl);
const payload: unknown = await response.json();
const delivery = deliverySchema.parse(payload);

this.context.set(delivery);
```

`status` и `trackingNumber` не входят в `schema` Context: их пишет сервер после проверки. Если те же поля должен менять и клиент, перенеси схему в `Context` и разбирай ответ через `context.schema.parse`.

## Антипаттерны

Клиент пишет ключ, которого нет в схеме:

```typescript
context = new Context({
  data: async () => ({ query: "", role: "viewer" }),
});

textInput({ ref: this.context.ref("query") });
```

`query` меняет клиент. Без `schema` входящий `context-values` не фильтруется по ключам: клиент может прислать и `query`, и `role` без Zod-проверки. В схему добавляй только `query`.

Серверные поля внешнего API добавлены в `schema` Context только ради `parse`. Тогда клиент тоже сможет их менять. Для таких полей нужна отдельная Zod-схема.

Ответ `fetch` записан в контекст с приведением типа, без `parse`. TypeScript тип HTTP-ответа не проверяет.

## Чеклист

- [ ] Данные клиента, стороннего API и других недоверенных источников проверены Zod до использования
- [ ] Каждый ref компонента, пишущего в контекст (input и аналоги), есть в `schema`
- [ ] Каждый путь `context.local` / `ref.local` (`set`, `toggle`, `inc`, `dec`) есть в `schema`
- [ ] Тип в схеме принимает значения, которые контрол или local action отправляет
- [ ] Вложенный путь описан объектом или массивом в схеме
- [ ] Ключи только BFF в `schema` не добавлены
- [ ] Поля внешнего ответа, которые клиент менять не должен, проверены отдельной схемой, а не через `schema` Context
- [ ] Контекст без клиентских записей оставлен без `schema`
- [ ] Секреты и данные, которые клиент не должен видеть, не лежат в сериализуемом контексте
- [ ] Бизнес-проверка формата осталась в server handler

## Связанные скиллы

- **`context-zod-draft-not-live-strict`** — мягкая схема на живом вводе, строгая проверка на submit
- **`instant-ui-set-context-value`** — `context.local` / `ref.local` и синхронизация `context-values`
- **`server-action-validate-in-handler`** — бизнес- и auth-проверки в обработчике, не в типах контекста
