---
name: webmcp-tool-name-and-input
description: >-
  Declares Matreshka WebMCP tools whose name contains only Latin letters,
  digits, hyphens, and underscores, and whose inputSchema is a Zod schema.
  Field descriptions explain UI choices such as select labels behind numeric
  ids. The execute return string and a thrown Error message are what the
  calling agent sees and must be understandable. Conditions only control
  visibility. Await platform command observables and context delivery before
  returning, including a chain that opens a dialog and then prefills it. Use
  when adding or editing webMcpTool, WebMcpTool, entry tools(), platform
  setTools, or execute results and errors.
---

# WebMCP-тулз

`webMcpTool({ ... })` и `new WebMcpTool({ ... })` — один класс, лучше использовать функциональный способ `webMcpTool`. `execute` остаётся на BFF и возвращает строку. Клиент отдаёт агенту `name`, `description` и JSON Schema из `inputSchema`.

## Имя

`name` после `trim` обязано совпасть с `/^[A-Za-z0-9_-]+$/`. Иначе конструктор падает с ошибкой: имя может содержать только латиницу, цифры, дефис и подчёркивание.

Разрешены `save_note`, `get-order-status`, `fillNote2`. Не проходят пробел внутри, кириллица, точка, слэш, пустая строка.

Агент выбирает тулз по имени. Имена одновременно доступных тулзов должны различаться: при коллизии клиент дописывает суффикс `save_2`, и по звучанию тулз уже хуже узнаётся.

## Input

Если тулз принимает аргументы, передай `inputSchema` схемой Zod 4. TypeScript выводит тип аргумента `execute` из этой схемы. BFF проверяет input до `execute`; при ошибке агент получает текст Zod, а `execute` не вызывается. Без `inputSchema` вызывай `execute` без аргументов.

В браузер уходит `z.toJSONSchema(inputSchema)`. Комментарий в коде агент не видит. Не подставляй готовый JSON Schema-объект: смысл поля пиши через `.describe()`.

### Описание полей

Перед схемой прочитай, что `execute` делает с input. Если значение пишется в `Context` (`context.set`, `ref.set`), найди остальные места, откуда то же поле туда попадает:

- стартовые `data()` и схема контекста;
- input, привязанный к тому же ref;
- `onClick`, submit и другие handler, которые делают `set` того же ключа;
- подписи, placeholder и варианты выбора рядом с контролом.

Тип Zod — это хранимое значение. В `.describe()` оставь только то, что нужно, чтобы его выбрать: подпись и допустимые варианты. Туда не входят Context, BFF и то, куда значение потом запишется.

Голого `z.number()` или `z.string()` для id недостаточно. Агент увидит «число» и подставит произвольный id.

Если набор значений известен в момент `tools()`, закрой его одним `z.literal` по id из того же массива, что рисует UI:

```ts
z.literal(options.map((item) => item.id));
```

В JSON Schema это `enum`. В `.describe()` перечисли пары «значение — подпись» из того же массива.

То же правило для других контролов: у `numberInput` укажи min, max, step и единицу; у текста — формат и что это не id; у даты — ISO с часовым поясом; у boolean — что означает `true` по подписи на экране.

`description` тулза говорит, зачем его вызывать. `.describe()` поля говорит, какое значение передать.

### Пример: селект, в контексте число

На экране селект «Тип доставки». В контекст и в тулз уходит `typeId`, подпись остаётся только в UI.

```ts
const deliveryTypes = [
  { id: 1, label: "Курьер" },
  { id: 2, label: "Самовывоз" },
  { id: 3, label: "Пункт выдачи" },
] as const;

// Тот же deliveryTypes питает селект на экране.
const typeIds = deliveryTypes.map((item) => item.id);
const typeIdOptions = deliveryTypes
  .map((item) => `${item.id} — «${item.label}»`)
  .join("; ");

webMcpTool({
  name: "set_delivery_type",
  description: "Выбирает тип доставки. Заказ не оформляет.",
  inputSchema: z.object({
    typeId: z.literal(typeIds).describe(`Тип доставки: ${typeIdOptions}.`),
  }),
  execute: async (input) => {
    await lastValueFrom(this.context.set({ typeId: input.typeId }));
    const label = deliveryTypes.find((item) => item.id === input.typeId)?.label;
    return `Выбран тип «${label}»`;
  },
});
```

`lastValueFrom` здесь ждёт всех получателей контекста. Это допустимо когда контекст существует только для одного клиента. Как дождаться конкретного клиента, описано ниже.

Так делать не нужно: имя с пробелом и кириллицей не создастся, а `typeId: z.number()` не говорит агенту, какое число какому пункту селекта соответствует.

```ts
webMcpTool({
  name: "Выбрать тип",
  description: "Ставит тип",
  inputSchema: z.object({
    typeId: z.number(),
  }),
  execute: async (input) => {
    this.context.set({ typeId: input.typeId });
    return "ok";
  },
});
```

## Условия видимости

`conditions` решают, виден ли тулз агенту. Пока они выполнены, клиент его показывает, когда перестают — убирает. Несколько условий в массиве означают «и».

BFF при вызове `conditions` не проверяет. Проверку на момент выполнения пиши в `execute`.

## Дождаться действия на клиенте

Перед `return` дождись действий, чей результат должен быть уже виден в браузере. Иначе агент получит ответ, пока экран ещё не изменился.

Команды клиентской платформы возвращают Observable. Из них собирай цепочку. `firstValueFrom` ждёт согласованную точку: диалог смонтирован, геолокация получена, скачивание запущено.

`context.set` и `ref.set` отдают прогресс доставки: в `delivered` копятся клиенты, которые уже получили значения. Для экрана одного пользователя дождись, пока в списке появится его `Client`. Чтобы дождаться всех получателей, используй `lastValueFrom`: он завершается, когда значения получили все.

```ts
const platform = currentClientPlatform();
const client = currentClient();
const dialog = new RequestDialog();

await firstValueFrom(platform.showDialog(dialog));
await firstValueFrom(
  dialog.context
    .set({ comment: input.comment })
    .pipe(filter(({ delivered }) => delivered.includes(client))),
);

return "Диалог заявки открыт, комментарий подставлен. Проверьте данные и сохраните.";
```

`showDialog` завершается, когда диалог смонтирован, а не когда пользователь его закрыл. После этого заполни его контекст и только потом отвечай агенту.

## Результат и ошибка execute

Строка из `return` — результат вызова, её читает агент. Напиши, что произошло: что создано, номер, статус. «ok», «true» и «done» агенту не помогают.

`throw new Error("...")` прерывает вызов. Агент видит текст ошибки. Он должен быть понятным, без технических подробностей. Вызов оберни в `try`, в `catch` проверь тип через `instanceof` и по нему собери этот текст. Если ошибку уже вернула команда платформы, отдай агенту её текст как есть.

```ts
webMcpTool({
  name: "create_request",
  description: "Создаёт заявку на доставку.",
  inputSchema: z.object({
    comment: z.string().describe("Комментарий к заявке."),
  }),
  execute: async (input) => {
    try {
      const request = await this.requests.create(input.comment);
      return `Заявка №${request.number} создана. Статус: «Новая».`;
    } catch (error) {
      if (error instanceof AddressOutsideZoneError) {
        throw new Error("Доставка по этому адресу недоступна.");
      }
      if (error instanceof SlotTakenError) {
        throw new Error("Выбранное время доставки уже занято.");
      }
      throw new Error("Не удалось создать заявку.");
    }
  },
});
```

## Чеклист

- [ ] `name` после trim — только `[A-Za-z0-9_-]`, среди видимых тулзов оно уникально
- [ ] Аргументы заданы Zod-схемой в `inputSchema`, не JSON Schema-объектом
- [ ] Если input пишется в контекст, описание взято из UI и других `set` того же поля
- [ ] У id и enum в описании есть пары «значение — подпись», набор закрыт `z.literal`
- [ ] В `.describe()` и `description` нет Context, BFF и прочего устройства записи
- [ ] У числа, строки, даты и boolean в описании есть формат и смысл на экране, не только тип
- [ ] Проверка на момент вызова написана в `execute`, не только в `conditions`
- [ ] Перед `return` дождался команд платформы и доставки контекста, если от них зависит экран
- [ ] Успешный `return` — строка о том, что произошло, а не «ok»
- [ ] Текст ошибки понятен агенту и без технических подробностей
