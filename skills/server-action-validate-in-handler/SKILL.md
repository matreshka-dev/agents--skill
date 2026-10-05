---
name: server-action-validate-in-handler
description: >-
  Matreshka BFF action conditions run only on the client; the client sends
  handlers field for dispatch. Use when adding ServerAction,
  action conditions, or reviewing auth and invariants in server handlers.
---

# Проверки в серверном обработчике (Matreshka BFF)

## Правило

`conditions` у действия (`LocalAction`, `ServerAction`, анимации и т.д.) проверяются **только на клиенте**. Они нужны, чтобы не слать лишние сообщения на сервер и не выполнять лишние клиентские действия, и **не являются безопасной стратегией проверки бизнес-условий**.

Проверки нужно добавлять в **серверный обработчик**, а не доверять контексту: контекст может быть неактуальным на момент срабатывания обработчика или, в случае нестрогих правил валидации контекста, быть изменённым злоумышленником.

## Диспетчеризация ServerAction

1. Клиент снимает `conditions` по исходному Context **до** выполнения actions в том же массиве.
2. Для каждого `server-interaction`, прошедшего фильтр, в сообщение добавляется его **индекс** в `interactions[event][]` (вместе с local actions в том же массиве).
3. BFF выполняет только `ServerAction` с индексами из `handlers`, **без** повторной проверки conditions.
4. Если ни один server-handler не прошёл фильтр, interaction на BFF не уходит.

Несколько `ServerAction` на одно событие с разными `conditions` (в том числе взаимоисключающими) — нормальный паттерн: клиент передаёт индексы только прошедших снимок handlers.

## Что делать в handler

1. Перед side effect (запись, оплата, удаление) проверяй права и инварианты через сервис/API с данными авторизации.
2. Не считай `this.context.value(...)` достаточным доказательством права на операцию.
3. Не полагайся на `conditions` action как на server gate — они только для клиентского снимка и экономии round-trip.

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

`conditions` на action уместны для UX (не слать interaction, пока `loading === true`), но не заменяют проверки в handler.

## Чеклист агента

- [ ] Бизнес- и auth-проверки в `ServerAction` / сервисе, не только в `conditions`
- [ ] Не полагаться на Context как на единственный gate для критичных операций
- [ ] Несколько `ServerAction` на событие — разные `conditions` на клиенте; guards в handler для критичных операций
- [ ] `conditions` у action — клиентская фильтрация и снимок, не server authorization

## Связанные скиллы

- **`prevent-duplicate-form-submit`** — `conditions` для `loading` на клиенте
- **`return-server-action-promise`** — ошибки async в UI
- **`context-zod-draft-not-live-strict`** — валидация ввода vs strict на submit
