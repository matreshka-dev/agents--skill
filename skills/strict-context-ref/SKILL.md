---
name: strict-context-ref
description: >-
  Types Matreshka BFF ContextRef strictly instead of ContextRef any any. Use
  when adding context refs, forEach generators, shared UI helpers taking refs,
  or fixing TS errors on ref value and nested ref paths.
---

# Строгая типизация `ContextRef` (Matreshka BFF)

## Зачем

`ContextRef<any, any>` отключает связь ref с полями контекста. TypeScript **не знает**, что лежит в `value()` и какие строки допустимы в `ref('…')` — появляются предупреждения, `any` в генераторах `forEach`, опечатки в путях не ловятся.

**По возможности** не используй `ContextRef<any, any>` в сигнатурах компонентов и хелперов BFF.

## Базовый случай: ref со страницы

`Context<T>` даёт строгий ref:

```typescript
this.context.ref('history'); // ContextRef<MyContext, 'history'>
this.context.ref('message'); // ContextRef<MyContext, 'message'>
```

Передавай такой ref в функции **без приведения к `any`**. Тип контекста страницы (`MyContext`) описывает поля явно.

## Общий компонент / хелпер

Если ref приходит параметром, зафиксируй **тип значения по пути**, а не `any`:

### Одно поле (строка, число, флаг)

```typescript
type MessageRef<C extends JsonObject, K extends Paths<C>> = ContextRef<C, K>;
// или проще для одного контекста:
type ComposerMessageRef = ContextRef<ChatRoomContext, 'message'>;
```

Можно обобщить компонент: `<C extends JsonObject>(messageRef: ContextRef<C, 'message'>)` — если поле всегда называется одинаково и есть constraint на `C`.

### Массив для `forEach`

Используй типы из **`forEach`**:

- **`ForEachDataRef<ItemType>`** — ref на массив элементов `ItemType`
- **`CompatibleForEachRef<…>`** — ref действительно указывает на массив (совместим с `forEach`)

```typescript
export type ChatHistoryListRef = CompatibleForEachRef<
  ForEachDataRef<ChatHistoryItem>
>;

export function chatHistoryList(historyRef: ChatHistoryListRef) { … }
```

В `generator: ({ ref: itemRef }) => …` тип элемента и вложенных `itemRef.ref('text')` выводится из `ItemType`.

### Вложенные поля элемента

Не аннотируй `{ ref: any }` — достаточно строгого ref списка; тип item ref приходит из **`ForEachGeneratorProperties`**.

## Полезные типы BFF

| Тип | Назначение |
|-----|------------|
| `ContextRef<C, P>` | Ref на поле `P` контекста `C` |
| `ContextRefValue<R>` | Тип **значения** по ref `R` |
| `ForEachDataRef<T>` | Ref на `T[]` |
| `CompatibleForEachRef<R>` | Ref на массив для `forEach` |

## Чеклист агента

- [ ] В новых параметрах нет `ContextRef<any, any>` без причины
- [ ] Для списков — `ForEachDataRef` + `CompatibleForEachRef` (или alias поверх них)
- [ ] Для полей страницы — ref из `context.ref('…')` или alias `ContextRef<MyContext, 'field'>`
- [ ] В `forEach` generator не типизировать ref как `any`
- [ ] Тип контекста страницы/диалога описан явно (`type XContext = { … }`), без лишнего `JsonObject &`

## Когда `any` ещё допустим

- Временный прототип с последующим сужением типа
- Граница с кодом, который **ещё** не типизирован (лучше сузить в следующем шаге)
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

Предпочитай **alias** (`ChatHistoryListRef`, `ComposerMessageRef`) в модуле компонента — call site остаётся `this.context.ref('history')`, проверка типов на стыке ref ↔ компонент.
