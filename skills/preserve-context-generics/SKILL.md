---
name: preserve-context-generics
description: >-
  Preserves Matreshka BFF Context and ContextRef generic types when passing them
  into functions—use function generics instead of bare Context, ContextRef,
  Context<any>, or ContextRef<any>. Use when adding helpers that take Context or
  ContextRef, fixing assignment or invariance errors, threading refs through a
  call chain, or reviewing function signatures.
---

# Context и ContextRef: сохранение типизации при передаче в функции

## Главное правило

При использовании `Context` или `ContextRef` в качестве аргументов функций всегда используй Generic-параметры функции, чтобы сохранить исходные типы контекста и пути.

Запрещено использовать `Context`, `ContextRef`, `Context<any>` или `ContextRef<any>` как универсальные способы принять произвольный экземпляр, если можно сохранить типизацию через Generic-параметры.

Если нужен только тип **значения** (`get()` / `set()`), а контекст и путь не важны — скиллы **[prefer-context-value-ref](../prefer-context-value-ref/SKILL.md)** (выбор типа: не целый `Context`, не фейковый `ContextRef`) и **[context-value-ref-in-api](../context-value-ref-in-api/SKILL.md)** (`get`/`set`, списки, компоненты). Этот скилл — когда нужно сохранить сами Generic-параметры `Context` / `ContextRef`.

## Почему это важно

`Context` и `ContextRef` — Generic-классы, типизация которых зависит от структуры данных контекста и, для `ContextRef`, от пути к конкретному значению.

Например:

```typescript
ContextRef<FinanceReportPageContext, "report">;
```

Содержит информацию:

- `FinanceReportPageContext` — тип исходного контекста.
- `"report"` — конкретный путь внутри контекста.

Если объявить функцию следующим образом:

```typescript
function processRef(ref: ContextRef) {
  // ...
}
```

TypeScript подставит Generic-параметры по умолчанию из объявления класса.

Например, класс объявлен так:

```typescript
class ContextRef<
  ContextType extends JsonObject = JsonObject,
  PathType extends string = string,
> {}
```

То `ContextRef` фактически означает:

```typescript
ContextRef<JsonObject, string>;
```

Это не универсальный тип для любых `ContextRef`.

Более того, из-за использования Generic-параметров внутри методов и свойств классы `Context` и `ContextRef` могут быть инвариантными. Поэтому экземпляр с конкретным типом контекста нельзя автоматически передать туда, где ожидается экземпляр с более общим типом.

Использование `any` может скрыть эту проблему, но приводит к потере строгой типизации.

## Правильный способ: Generic-параметры функции

### ContextRef

Неправильно:

```typescript
function processRef(ref: ContextRef) {
  return ref.get();
}
```

Неправильно:

```typescript
function processRef(ref: ContextRef<any>) {
  return ref.get();
}
```

Правильно:

```typescript
function processRef<T extends JsonObject, P extends string>(
  ref: ContextRef<T, P>,
) {
  return ref.get();
}
```

TypeScript самостоятельно выведет `T` и `P` из переданного экземпляра.

При этом сохраняется тип значения, возвращаемого `ref.get()`.

### Context

Аналогичное правило действует для `Context`. Фактическое объявление класса:

```typescript
class Context<T extends JsonObject> {}
```

Неправильно:

```typescript
function processContext(context: Context) {
  // ...
}
```

Неправильно:

```typescript
function processContext(context: Context<any>) {
  // ...
}
```

Правильно:

```typescript
function processContext<T extends JsonObject>(context: Context<T>) {
  // ...
}
```

Здесь `T` сохраняет тип данных контекста, а `S` — тип Zod-схемы.

При написании сигнатуры необходимо использовать фактическое объявление Generic-параметров класса `Context`, включая их ограничения и значения по умолчанию.

## Передача ContextRef между несколькими функциями

При передаче ссылки через цепочку функций сохраняй Generic-параметры на каждом уровне.

```typescript
function getReportRef<T extends JsonObject, P extends string>(
  ref: ContextRef<T, P>,
) {
  return processReportRef(ref);
}

function processReportRef<T extends JsonObject, P extends string>(
  ref: ContextRef<T, P>,
) {
  return ref.get();
}
```

Не превращай типизированную ссылку в `ContextRef` или `ContextRef<any>` при промежуточной передаче.

## Сохранение конкретного пути

Если функция работает только с определённым путём, дополнительно ограничивай Generic-параметр пути.

Например:

```typescript
function processReportRef<
  T extends FinanceReportPageContext,
  P extends Paths<T>,
>(ref: ContextRef<T, P>) {
  // ...
}
```

Если функция должна принимать только ссылку на `report`, можно использовать конкретный путь:

```typescript
function processReportRef<T extends FinanceReportPageContext>(
  ref: ContextRef<T, "report">,
) {
  // ...
}
```

Выбирай ограничения в соответствии с реальными требованиями функции, не ослабляя типизацию без необходимости.

## Правила для ИИ-агента

При разработке и изменении кода Matreshka:

1. Всегда проверяй, не используются ли `Context` и `ContextRef` без Generic-параметров в аргументах функций.
2. Если функция должна принимать экземпляры с произвольными типами контекста, объявляй Generic-параметры самой функции.
3. Для `ContextRef` сохраняй оба параметра: тип контекста и тип пути (`T`, `P`).
4. Для `Context` сохраняй Generic-параметры, определённые в объявлении класса.
5. Не заменяй конкретные Generic-типы на `any` ради устранения ошибок TypeScript.
6. Не используй приведение типов (`as any`, `as ContextRef<any>`, `as Context<any>`) для обхода несовместимости.
7. Если функция принимает только определённый тип контекста или путь, используй соответствующие ограничения Generic-параметров.
8. При обнаружении ошибки совместимости сначала исправляй сигнатуру функции, а не ослабляй типизацию передаваемого объекта.

## Исключение

Использование `Context<any>` или `ContextRef<any, any>` допустимо только в действительно динамических инфраструктурных участках, где конкретный тип контекста принципиально неизвестен и его сохранение невозможно. Такое решение должно быть осознанным, а не способом исправления ошибки компиляции.

## Итог

`Context` и `ContextRef` должны передаваться между функциями с сохранением исходных Generic-параметров.

Вместо попытки принять любой экземпляр через базовый тип или `any` используй Generic-параметры функции. Это позволяет сохранять строгую типизацию контекста, путей, возвращаемых значений и доступных операций на протяжении всей цепочки вызовов.

## Связанные скиллы

- [strict-context-ref](../strict-context-ref/SKILL.md) — `ContextRef<C, P>` на странице и в `forEach`, без `any`
- [prefer-context-value-ref](../prefer-context-value-ref/SKILL.md) — не тащить целый `Context` и не выдумывать `ContextRef<{ field: T }, "field">` без нужды
- [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md) — `ContextValueRef` / `ContextArrayRef`, когда важен только тип значения
