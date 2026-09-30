---
name: prevent-duplicate-form-submit
description: >-
  Protects Matreshka BFF forms from duplicate submissions with a Context
  loading flag, an immediate setContextValue action, and a conditional
  ServerAction. Use when creating, editing, or reviewing forms that send
  requests or start asynchronous operations.
---

# Защита формы от повторной отправки (Matreshka BFF)

## Правило

Для формы, которая отправляет запрос или запускает асинхронную операцию:

1. Храни в `Context` булевый флаг отправки, например `loading`, с начальным значением `false`.
2. При подтверждении сразу устанавливай флаг в `true` через клиентский `setContextValue`.
3. Добавляй серверному действию условие `when.equals(loadingRef, false)`.
4. После завершения операции возвращай флаг в `false`, включая сценарий ошибки.

Условие обязательно должно находиться на `ServerAction`: визуальная смена состояния кнопки сама по себе не предотвращает повторный запрос.

## Шаблон

```typescript
const loadingRef = this.context.ref("loading");

const submitForm = [
  setContextValue(loadingRef, true),
  new ServerAction(
    async () => {
      try {
        await signIn(
          this.context.value("login"),
          this.context.value("password"),
        );
      } finally {
        this.context.setValue("loading", false);
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

Клиент сначала отбирает действия по условиям, а затем выполняет их. Поэтому при первой отправке `ServerAction` проходит условие по исходному `loading === false`, даже если соседний `setContextValue` сразу меняет флаг на `true`. При следующей попытке условие уже ложно, и повторный запрос не отправляется.

## Не делать

- Не полагайся только на изменение текста, видимости или доступности кнопки.
- Не запускай серверную операцию без условия по флагу отправки.
- Не устанавливай флаг только внутри серверного callback: клиент не получит мгновенную защиту.
- Не забывай сбрасывать флаг при ошибке, иначе форма останется заблокированной.

## Чеклист агента

- [ ] В `Context` есть булевый флаг отправки со значением `false` по умолчанию
- [ ] Первое действие подтверждения — `setContextValue(flagRef, true)`
- [ ] `ServerAction` имеет `conditions: [when.equals(flagRef, false)]`
- [ ] Флаг сбрасывается после успеха и ошибки
- [ ] Один список действий используется в `form.onSubmit` и `button.onClick`
- [ ] Поля и кнопка находятся внутри `form`

## Связанный скилл

- [form-submit-shared-handler](../form-submit-shared-handler/SKILL.md) — общий сценарий подтверждения для `onSubmit` формы и `onClick` кнопки
