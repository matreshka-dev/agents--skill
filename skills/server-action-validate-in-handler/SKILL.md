---
name: server-action-validate-in-handler
description: >-
  Matreshka BFF action conditions run only on the client to skip local actions
  and avoid extra server messages; they are not a secure business-rule gate.
  Use when adding ServerAction, action conditions, or reviewing auth and
  invariants in server handlers.
---

# Проверки в серверном обработчике (Matreshka BFF)

## Правило

`conditions` у действия (`LocalAction`, `ServerAction`, анимации и т.д.) проверяются **только на клиенте**. Они нужны только для того, чтобы не слать лишние сообщения на сервер и не выполнять лишние клиентские действия, и **не являются безопасной стратегией проверки бизнес-условий**.

Проверки нужно добавлять в **серверный обработчик**, а не доверять контексту: контекст может быть неактуальным на момент срабатывания обработчика или, в случае нестрогих правил валидации контекста, быть изменённым злоумышленником.

## Что делать в handler

1. Перед side effect (запись, оплата, удаление) проверяй права и инварианты через сервис/API с данными авторизации.
2. Не считай `this.context.value(...)` достаточным доказательством права на операцию.
3. Если на одно событие несколько `ServerAction` с взаимоисключающими `conditions`, помни: BFF вызывает **все** handler'ы типа события — в каждом handler нужен явный guard или одна объединённая ветка.

## Шаблон

```typescript
new ServerAction(async () => {
  const userId = this.session.userId(); // auth источник
  const orderId = this.context.value("orderId");

  const allowed = await this.orders.canCancel(userId, orderId);
  if (!allowed) {
    return;
  }

  await this.orders.cancel(orderId);
});
```

`conditions` на action по-прежнему уместны для UX (не слать interaction, пока `loading === true`), но дублируют **не** security — только оптимизацию клиента.

## Чеклист агента

- [ ] Бизнес- и auth-проверки в `ServerAction` / сервисе, не только в `conditions`
- [ ] Не полагаться на Context как на единственный gate для критичных операций
- [ ] Несколько `ServerAction` на одно событие — guard в каждом handler или один handler
- [ ] `conditions` у action описаны как клиентская фильтрация, не как server authorization

## Связанные скиллы

- **`prevent-duplicate-form-submit`** — `conditions` для `loading` на клиенте
- **`return-server-action-promise`** — ошибки async в UI
- **`context-zod-draft-not-live-strict`** — валидация ввода vs strict на submit
