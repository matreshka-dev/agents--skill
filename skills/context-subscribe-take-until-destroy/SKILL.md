---
name: context-subscribe-take-until-destroy
description: >-
  Subscribes to Matreshka BFF Context watch() on entry components with
  takeUntil(destroy$) to avoid leaks after Page or Dialog unmount. Use when
  adding context subscriptions, side effects, or logging on context changes.
---

# Подписка на `context.watch()` и `destroy$` (Matreshka BFF)

## Правило

На **`Page`**, **`Dialog`**, **`Popover`** (entry) подписки на `this.context.watch()` должны завершаться вместе с entry:

```typescript
import { takeUntil } from "rxjs";

override async boot() {
  await super.boot();

  this.context
    .watch()
    .pipe(takeUntil(this.destroy$))
    .subscribe((data) => {
      // реакция на изменение контекста
    });
}
```

`watch()` ждёт `init()`, если хранилище ещё не создано. Без отписки — лишние реакции, утечки, обработка уже закрытого UI.

## Альтернатива

Предпочитать **`ref` + `conditions` / `rules`** и server handlers вместо длинных подписок, если задача только в UI.

## Антипаттерн

```typescript
// ❌ Подписка без takeUntil на entry
this.context.watch().subscribe(() => {
  this.doSomething();
});
```

## Чеклист

- [ ] Подписка только на entry с `destroy$`
- [ ] `takeUntil(this.destroy$)` или явная отписка в `onLeave`/destroy hook
- [ ] **`context-destroy-with-entry`** — уничтожение самого Context
