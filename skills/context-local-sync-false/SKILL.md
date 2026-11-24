---
name: context-local-sync-false
description: >-
  Uses Matreshka BFF context.local / ref.local with { sync: false } when the
  server does not need the client value. Client-only UI state (hover, form
  loading locks) updates the client mirror only—no context-values spam—and
  stays out of Context Zod schema; the server may still push new values to the
  client via set. Use when adding or reviewing local context writes, ephemeral
  UI flags, or duplicate-submit loading flags.
---

# `{ sync: false }` для local-записи Context (Matreshka BFF)

## Правило

`context.local.*` / `ref.local.*` по умолчанию сразу обновляют зеркало на клиенте
и шлют то же изменение на BFF сообщением `context-values`. Путь тогда обязан
быть в `schema` контекста.

Передавай `{ sync: false }`, когда **серверу неважно текущее клиентское
значение**:

- ephemeral UI (hover, подсветка пункта меню, локальный highlight);
- клиентский lock формы (`loading`), который блокирует повторный submit, пока
  сервер не сбросит флаг в handler.

С `{ sync: false }`:

- клиент пишет **только** зеркало и **не** отправляет `context-values`;
- ключ **не** добавляй в Zod `schema` — patch на BFF не уходит;
- TypeScript требует `{ sync: false }` для пути вне схемы;
- сервер **по-прежнему** может писать тот же ключ (`ref.set` / `context.set`)
  и доставлять новое значение клиенту.

Если BFF должен знать или валидировать значение с клиента (фильтры, черновики,
счётчики, которые читает серверный handler) — **не** ставь `sync: false`: путь
в `schema`, sync по умолчанию.

## Hover / UI-состояние

```typescript
stack(
  {
    onMouseEnter: this.context.local.set(
      { hoveredMenuItem: "accruals" },
      { sync: false },
    ),
    onMouseLeave: this.context.ref("hoveredMenuItem").local.set(null, {
      sync: false,
    }),
  },
  [text("Меню")],
);
```

Серверу hover не нужен: без `{ sync: false }` каждый `mouseenter`/`mouseleave`
спамил бы `context-values`. Ключ `hoveredMenuItem` в `schema` не описывают.

## Loading при submit

```typescript
const loadingRef = this.context.ref("loading");

const submitForm = [
  loadingRef.local.set(true, { sync: false }),
  new ServerAction(
    async () => {
      try {
        await submit();
      } finally {
        loadingRef.set(false);
      }
    },
    { conditions: [when.equals(loadingRef, false)] },
  ),
];
```

`loading` нужен клиенту, чтобы сразу заблокировать повторную отправку. Сервер
не читает клиентский `true` из `context-values`; он сам сбрасывает флаг через
`loadingRef.set(false)`. В `schema` ключ `loading` не включают.

Полный паттерн формы — **`prevent-duplicate-form-submit`**.

## Антипаттерны

```typescript
// ❌ Hover с sync по умолчанию — лишний трафик на BFF
onMouseEnter: this.context.local.set({ hovered: true });

// ❌ Client-only loading в schema — ключ открыт для входящего context-values
schema: z.object({ loading: z.boolean(), login: z.string() });
loadingRef.local.set(true); // без sync: false
```

## Чеклист

- [ ] Серверу не нужно текущее значение с клиента → `{ sync: false }`
- [ ] Ключ с `{ sync: false }` **не** в `schema`
- [ ] При необходимости сброса/обновления с BFF — `ref.set` / `context.set` (server push ок)
- [ ] Значение нужно handler'у или должно валидироваться на BFF → sync по умолчанию + ключ в `schema`
- [ ] Hover / spam-prone UI → `{ sync: false }`
- [ ] Loading lock формы → `{ sync: false }` + сброс на сервере; см. **`prevent-duplicate-form-submit`**

## Связанные скиллы

- **`instant-ui-set-context-value`** — API `context.local` / `ref.local`
- **`context-validate-untrusted-sources`** — что класть в `schema`
- **`prevent-duplicate-form-submit`** — loading + conditional `ServerAction`
- **`writable-context-ref-for-inputs-and-local-actions`** — типы ref: sync-on vs `{ sync: false }`
