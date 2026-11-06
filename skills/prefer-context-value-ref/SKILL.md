---
name: prefer-context-value-ref
description: >-
  Chooses Matreshka BFF ContextValueRef or ContextArrayRef for function
  parameters when only ref value type matters—not full context shape. Avoids
  Context<WholePageContext>, fake ContextRef<{ field: T }, "field">, and
  generic C extends JsonObject only to reach one path. Use when adding shared
  helpers, refactors, or deciding ContextRef vs ContextValueRef; use
  preserve-context-generics when whole context or path typing is required.
---

# `ContextValueRef`, когда структура контекста не важна

## Главное правило

Если функции нужна **ссылка на поле** и достаточно знать тип **`get()` / `set()`**, остальные ключи контекста **не фиксируй**: параметр — **`ContextValueRef<T>`** или **`ContextArrayRef<Item>`** для списков.

Используй **`Context`** / **`ContextRef<C, P>`** только когда важна **структура контекста** и/или **вложенные пути** `ref('a.b')` от того же экземпляра — тогда см. **[preserve-context-generics](../preserve-context-generics/SKILL.md)**.

Терминология: в BFF тип параметра — **`ContextValueRef<T>`**. **`ContextRefValue<R>`** — вспомогательный тип «значение по ref `R`», не замена для сигнатуры (см. `@matreshka/bff/core`).

Подробнее про `get`/`set`, `CompatibleForEachRef`, компоненты ввода — **[context-value-ref-in-api](../context-value-ref-in-api/SKILL.md)**.

## Когда какой тип

| Нужно функции | Тип параметра |
| ------------- | ------------- |
| Ссылка на значение типа `T`, остальные поля контекста не важны | `ContextValueRef<T>` |
| Ссылка на массив `T[] \| undefined` для списка / `forEach` | `ContextArrayRef<T>` |
| Несколько полей одного контекста, соседние пути, вложенный `ref` от **реального** `C` | `Context<C>` или `ContextRef<C, P>` (+ generics) |

## Чего не делать

- **Не** принимай `Context<WholeDialogOrPageValues>`, если функции достаточно **одного** поля — передай **`ContextValueRef`** на это поле (или вызови с `this.context.ref('…')`).
- **Не** объявляй «универсальный ref на T» как фиктивный контекст:

  ```typescript
  ContextRef<{ amount: number }, "amount">;
  ```

  Это обман TS: реальный контекст другой; на call site будет лишний каст или ложное чувство типобезопасности.

- **Не** заводи `C extends JsonObject`, `P extends SomePath<C>` **только** чтобы пробить ref на один тип значения — если путь и имя поля на странице могут отличаться, это **`ContextValueRef`**, не generic по всему контексту.

## Простые примеры

### Фейковый контекст ради имени поля

Неправильно:

```typescript
type AmountRef = ContextRef<{ amount: number }, "amount">;

function amountField(ref: AmountRef) {}
```

Правильно:

```typescript
import type { ContextValueRef } from "@matreshka/bff/core";

function amountField(ref: ContextValueRef<number | undefined>) {}
```

### Весь `Context` ради одного поля

Неправильно:

```typescript
function toggleDraft(context: Context<DialogValues>, roleId: number) {
  const ids = context.get("draftSelectedIds");
  context.set({
    draftSelectedIds: /* … */,
  });
}
```

Правильно:

```typescript
function toggleDraft(
  selectedIdsRef: ContextValueRef<number[]>,
  roleId: number,
) {
  const ids = selectedIdsRef.get();
  selectedIdsRef.set(/* … */);
}
```

### Узкий union путей страницы при том же типе значения

Неправильно (хелперу достаточно `boolean`):

```typescript
function sectionToggle(
  expandedRef: ContextRef<DetailsPage, "blockAExpanded" | "blockBExpanded">,
) {
  expandedRef.local.toggle();
}
```

Правильно:

```typescript
function sectionToggle(expandedRef: ContextValueRef<boolean | undefined>) {
  expandedRef.local.toggle();
}
```

### Generic по контексту ради списка

Неправильно:

```typescript
function cards<C extends JsonObject, P extends PathToCardList<C>>(
  listRef: ContextRef<C, P>,
) {}
```

Правильно:

```typescript
import type { ContextArrayRef } from "@matreshka/bff/core";

type Card = { id: string; title: string };

function cards(listRef: ContextArrayRef<Card>) {}
```

## Когда `ContextRef` уместен

- Хелпер строит UI от **нескольких** ref из одного сценария с проверкой имён полей.
- Нужны **вложенные** `ref('nested.field')`, типизированные от **реального** `ContextType` страницы/диалога.
- В **`forEach`** generator — ref строки с типом элемента (см. **[strict-context-ref](../strict-context-ref/SKILL.md)**).

## Чеклист агента

- [ ] Параметр «ref на значение T» — **`ContextValueRef<T>`**, не выдуманный `ContextRef<{ … }, "…">`.
- [ ] Список — **`ContextArrayRef<Item>`**, не `ContextRef<C, P>` + generic по путям, если важен только тип элемента.
- [ ] Не тащи **весь** `Context` в shared-хелпер ради одного `get`/`set` — передай ref на поле.
- [ ] Если снова нужны `C` и `P` — сначала проверь, нельзя ли заменить на `ContextValueRef`; если нет — **[preserve-context-generics](../preserve-context-generics/SKILL.md)**.

## Связанные скиллы

- [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md) — паттерны `get`/`set`, `CompatibleForEachRef`, инпуты
- [preserve-context-generics](../preserve-context-generics/SKILL.md) — когда нужны именно `Context` / `ContextRef` с generics
- [strict-context-ref](../strict-context-ref/SKILL.md) — строгий `ContextRef` на странице и в `forEach`
