---
name: emit-after-in-place-mutation
description: >-
  Calls emitAfterInPlaceMutation after mutating Matreshka BFF Context data
  in place; prefers setValue for normal updates. Use when reviewing in-place
  array or object mutations on context data$.
---

# In-place mutation и `emitAfterInPlaceMutation` (Matreshka BFF)

## Правило

Предпочтительно:

- `this.context.setValue(path, value)`;
- неизменяемое обновление: `[...items, newItem]`.

In-place mutation объекта из `data$!.getValue()` — **bad practice**, но если применили:

```typescript
const data = this.context.data$!.getValue();
data.items.push({ id: "3", title: "Новый" });
this.context.emitAfterInPlaceMutation();
```

Без `emitAfterInPlaceMutation()` подписчики и клиент **не** увидят изменение.

`setValue()` публикует обновление сам.

## Чеклист

- [ ] По возможности заменить на `setValue` / spread
- [ ] После ручной мутации — `emitAfterInPlaceMutation()`
- [ ] Списки в UI — **`foreach-stable-track`**
