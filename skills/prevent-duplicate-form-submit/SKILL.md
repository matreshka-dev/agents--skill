---
name: prevent-duplicate-form-submit
description: >-
  Protects Matreshka BFF forms from duplicate submissions with a Context
  loading flag, an immediate ref.local.set(..., { sync: false }) action, and a
  conditional ServerAction. Use when creating, editing, or reviewing forms that
  send requests or start asynchronous operations.
---

# Защита формы от повторной отправки (Matreshka BFF)

## Правило

Для формы, которая отправляет запрос или запускает асинхронную операцию:

1. Храни в `Context` булевый флаг отправки, например `loading`, с начальным значением `false`.
2. При подтверждении сразу устанавливай флаг в `true` через `loadingRef.local.set(true, { sync: false })` (или `context.local.set({ loading: true }, { sync: false })`).
3. Добавляй серверному действию условие `when.equals(loadingRef, false)`.
4. После завершения операции возвращай флаг в `false` на BFF (`loadingRef.set(false)`), включая сценарий ошибки.

`loading` нужен только клиенту для мгновенной блокировки; серверу клиентский `true` не нужен. `{ sync: false }` не шлёт `context-values`, поэтому ключ **не** включай в Zod `schema`. Сервер по-прежнему сбрасывает флаг через `ref.set` — см. **`context-local-sync-false`**.

Условие на `ServerAction` нужно для **клиента**: без него interaction уйдёт на BFF повторно. Визуальная смена состояния кнопки сама по себе не предотвращает повторный запрос. BFF не повторяет `conditions` — для идемпотентности и auth см. **`server-action-validate-in-handler`**.

## Шаблон

```typescript
const loadingRef = this.context.ref("loading");

const submitForm = [
  loadingRef.local.set(true, { sync: false }),
  new ServerAction(
    async () => {
      try {
        await signIn(
          this.context.get("login"),
          this.context.get("password"),
        );
      } finally {
        loadingRef.set(false);
      }
    },
    {
      conditions: [when.equals(loadingRef, false)],
    },
  ),
];

return form(
  {
    onSubmit: submitForm,
  },
  [
    textInput(this.context.ref("login")),
    passwordInput(this.context.ref("password")),
    button(
      {
        onClick: submitForm,
        rules: [
          {
            conditions: [when.equals(loadingRef, true)],
            overrides: {
              content: [text("Отправляем...")],
            },
          },
        ],
      },
      [text("Отправить")],
    ),
  ],
);
```

Клиент сначала отбирает действия по условиям, а затем выполняет их. Поэтому при первой отправке `ServerAction` проходит условие по исходному `loading === false`, даже если соседний `loadingRef.local.set(true, { sync: false })` сразу меняет флаг. При следующей попытке условие уже ложно, и повторный запрос не отправляется.

## Не делать

- Не полагайся только на изменение текста, видимости или доступности кнопки.
- Не запускай серверную операцию без условия по флагу отправки.
- Не устанавливай флаг только внутри серверного callback: клиент не получит мгновенную защиту.
- Не забывай сбрасывать флаг при ошибке, иначе форма останется заблокированной.
- Не клади `loading` в `schema` и не синхронизируй его на BFF без нужды: используй `{ sync: false }`.

## Чеклист агента

- [ ] В `Context` есть булевый флаг отправки со значением `false` по умолчанию
- [ ] Первое действие подтверждения — `flagRef.local.set(true, { sync: false })`
- [ ] Ключ `loading` **не** в Zod `schema`
- [ ] `ServerAction` имеет `conditions: [when.equals(flagRef, false)]`
- [ ] Флаг сбрасывается после успеха и ошибки (`loadingRef.set(false)` в `finally`, не дублируй путь через `context.set`)
- [ ] Один список действий используется в `form.onSubmit` и `button.onClick`
- [ ] Поля и кнопка находятся внутри `form`

## Связанные скиллы

- **`form-submit-shared-handler`** — общий handler для `onSubmit` и `onClick`
- **`context-local-sync-false`** — когда `{ sync: false }` и ключ вне schema
- **`instant-ui-set-context-value`** — зачем первым идёт local set на клиенте
- **`context-value-ref-in-api`** — запись через `ref.set`, когда ref уже в scope
- **`rules-not-conditions-for-loading-ui`** — текст «Отправляем…» через `rules`
