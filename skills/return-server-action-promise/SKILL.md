---
name: return-server-action-promise
description: >-
  Ensures Matreshka BFF ServerAction and server event handlers return or await
  asynchronous work so rejected promises are routed to client.error$. Use when
  adding or reviewing async onClick, onSubmit, lifecycle handlers, or explicit
  ServerAction callbacks.
---

# Возврат Promise из ServerAction (Matreshka BFF)

## Правило

ServerAction перехватывает reject только у promise, который вернул обработчик. Синхронный throw и await внутри async-функции тоже попадают в client.error$, а не в unhandledRejection.

Не оборачивайте async-вызов во внутренний void — снаружи обработчик тогда возвращает undefined, и BFF не видит promise:

```typescript
// ❌ Promise скрыт: reject не попадёт в client.error$
onClick: () => {
  void this.save();
};

// ✅ Promise возвращён обработчиком
onClick: () => this.save();

// ✅ async-обработчик ожидает Promise и возвращает свой Promise
onClick: async () => {
  await this.save();
};
```

То же правило действует для явного `ServerAction`:

```typescript
// ❌ execute() получает undefined
new ServerAction(() => {
  void this.save();
});

// ✅ execute() получает Promise
new ServerAction(() => this.save());

// ✅ reject от await станет reject возвращённого Promise
new ServerAction(async () => {
  await this.save();
});
```

## Почему это важно

`ServerAction`:

1. перехватывает синхронный `throw` из обработчика;
2. проверяет возвращённое значение;
3. если это Promise, подписывается на его `catch`;
4. передаёт ошибку в `client.error$`.

Вызов `void this.save()` запускает Promise, но отбрасывает его. Обработчик возвращает `undefined`, поэтому BFF не может связать последующий reject с `ServerAction`.

## Допустимый `void`

Не используй `void` для асинхронной операции, ошибку которой должен обработать `ServerAction`. Он допустим только для намеренного fire-and-forget, если вызываемый код сам полностью обрабатывает reject:

```typescript
onClick: () => {
  void this.runInBackground().catch((error) => {
    this.backgroundErrors.report(error);
  });
};
```

По умолчанию предпочитай возврат Promise, а не fire-and-forget.

## Чеклист агента

- [ ] Async-вызов возвращается напрямую или ожидается через `await`
- [ ] Обработчик не скрывает Promise через внутренний `void`
- [ ] `async`-обработчик не запускает незавершённую Promise-цепочку без `await`
- [ ] Синхронный `throw` и reject остаются доступны механизму `client.error$`
- [ ] Fire-and-forget используется только с собственным обработчиком ошибки
