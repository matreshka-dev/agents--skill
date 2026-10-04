---
name: strict-context-ref
description: >-
  Types Matreshka BFF ContextRef with context and path on pages—avoid ContextRef
  any any, explicit page context types, forEach generator refs. Use when
  defining Page/Dialog context, context.ref paths, or nested ref in lists. For
  shared function parameters by value type only, use context-value-ref-in-api.
---

# Строгая типизация `ContextRef` (Matreshka BFF)

## Зачем

`ContextRef<any, any>` отключает связь ref с полями контекста. TypeScript **не знает**, что лежит в `value()` и какие строки допустимы в `ref('…')` — опечатки в путях не ловятся, в `forEach` пропадает тип элемента.

**Не используй `ContextRef<any, any>`** в публичных сигнатурах без веской причины.

> **Shared-хелперы и методы**, где важен только тип значения, а не контекст/путь — см. скилл **[context-value-ref-in-api](../context-value-ref-in-api/SKILL.md)** (`ContextValueRef`, `ContextArrayRef`).

## Базовый случай: ref со страницы

`Context<T>` даёт строгий ref:

```typescript
this.context.ref('history'); // ContextRef<MyContext, 'history'>
this.context.ref('message'); // ContextRef<MyContext, 'message'>
```

Передавай такой ref **без приведения к `any`**. Тип контекста страницы (`MyContext`) описывает поля явно.

## Когда нужен именно `ContextRef<C, P>`

- Описание поля на **конкретной** странице или alias `ContextRef<ChatRoomContext, 'message'>`
- Generic с constraint: `<C extends JsonObject>(ref: ContextRef<C, 'message'>)` — если имя поля фиксировано во всех `C`
- В **`forEach` generator** тип item ref выводится из ref списка; не подменяй `{ ref: any }`

## Массивы и `forEach`

На **call site** ref списка остаётся строгим: `this.context.ref('messages')`.

В **сигнатуре shared-хелпера** — `ContextArrayRef<ItemType>` (скилл **context-value-ref-in-api**).

В конфиге `forEach`:

- **`CompatibleForEachRef<R>`** — ref действительно указывает на массив

```typescript
new ForEach({
  ref: this.context.ref('messages'),
  track: (message) => message.id,
  generator: ({ ref: itemRef }) => {
    itemRef.ref('text'); // пути проверяются по типу элемента
  },
});
```

## Полезные типы BFF

| Тип | Назначение |
|-----|------------|
| `ContextRef<C, P>` | Ref на поле `P` контекста `C` (страница, dialog) |
| `ContextRefValue<R>` | Тип **значения** по ref `R` |
| `ContextValueRef<V>` | Ref с известным `V` без фиксации `C`/`P` — **сигнатуры API** |
| `ContextArrayRef<T>` | Ref на `T[] \| undefined` |
| `CompatibleForEachRef<R>` | Call site: ref на массив для `forEach` |
| `CompatibleContextValueRef<V, R>` | Call site: ref `R` совместим со значением `V` |

## Чеклист агента

- [ ] Нет `ContextRef<any, any>` в новых параметрах без причины
- [ ] Shared-функция по **типу значения** — `ContextValueRef` / `ContextArrayRef`, не этот скилл вместо того
- [ ] Тип контекста страницы/диалога явный (`type XContext = { … }`)
- [ ] В `forEach` generator не типизировать ref как `any`
- [ ] Списки на call site — `context.ref('…')`; при необходимости `CompatibleForEachRef`

## Когда `any` ещё допустим

- Временный прототип с последующим сужением
- Внутренности фреймворка BFF, не публичные API приложения

## Антипаттерн

```typescript
function userMultiSelect(usersRef: ContextRef<any, any>, …) {
  forEach({
    generator: ({ ref: userRef }: { ref: any }) => {
      userRef.ref('displayName'); // TS не проверяет путь
    },
  });
}
```

## Ориентир

На странице — **alias** `ContextRef<MyContext, 'field'>` или вывод из `context.ref('…')`. В общем API — **context-value-ref-in-api**.
