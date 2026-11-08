---
name: writable-context-ref-for-inputs-and-local-actions
description: >-
  Uses WritableContextRefOfValue for input refs and context local actions.
  Prevents passing read-only paths to set/toggle/inc/dec and keeps strict typing
  for form fields and instant UI updates without a generic over the ref.
---

# WritableContextRefOfValue для инпутов и local-действий Context (Matreshka BFF)

## Правило

Если ref используется для записи (input binding, `set`, `toggle`, `inc`, `dec`), путь должен быть не только совместим по типу значения, но и разрешён в writable schema.

- Для сигнатур «по значению» без записи используй `ContextRefOfValue<T>`.
- Для сигнатур записи используй `WritableContextRefOfValue<T>`.

`WritableContextRefOfValue<T>` не требует generic по ref (`R`): writable-принадлежность выражена структурно (`local.set` и другие `local.*` есть только у writable-путей), а `local` выводится из `ContextRef.local`. Ref с путём вне writable-путей schema не пройдёт проверку, новые методы `local` подхватываются автоматически.

## Когда какой тип использовать

| Сценарий                                                                | Тип                            |
| ----------------------------------------------------------------------- | ------------------------------ |
| Нужен ref по типу значения, запись не обязательна                       | `ContextRefOfValue<T>`         |
| Нужен ref, который точно можно менять из клиента (input, `ref.local.*`) | `WritableContextRefOfValue<T>` |

## Базовые примеры

```typescript
import type {
  ContextRefOfValue,
  WritableContextRefOfValue,
} from "@matreshka/bff/core";

// Только работа со значением
function readOrRenderAmount(ref: ContextRefOfValue<number | undefined>) {
  return ref.get();
}

// Запись только в writable-путь
function bindWritableAmount(
  ref: WritableContextRefOfValue<number | undefined>,
) {
  ref.set(0);
}
```

## Инпуты: строгая типизация записи

Для input-компонентов ref должен быть writable, иначе значение нельзя безопасно отправлять в `context-values`.

```typescript
function amountInput(ref: WritableContextRefOfValue<number | undefined>) {
  return input.number({ ref });
}
```

Если нужен только read-only вывод в UI, используй output и обычный `ContextRefOfValue<T>`.

## Local-действия Context: только writable пути

`ref.local.set(...)`, `ref.local.toggle()`, `ref.local.inc(...)`, `ref.local.dec(...)` вызываются на writable ref.

```typescript
function finishSubmit(
  loadingRef: WritableContextRefOfValue<boolean | undefined>,
) {
  return loadingRef.local.set(false);
}
```

## Антипаттерны

```typescript
// ❌ Совместимость по value есть, но путь может быть read-only
function badToggle(ref: ContextRefOfValue<boolean | undefined>) {
  ref.local.toggle();
}

// ❌ Стирание типов ломает проверку writable-пути
function badInput(ref: any) {
  return input.text({ ref });
}

// ❌ Лишний generic по ref и устаревший тип
function oldStyle<R extends AnySchemaContextRef>(
  ref: CompatibleWritableContextRefOfValue<number | undefined, R>,
) {}
```

## Чеклист агента

- [ ] Для input/ref с записью используется `WritableContextRefOfValue<T>`
- [ ] Нет `any` в сигнатурах ref для форм и local-действий
- [ ] Для read-only сценариев не навязывается writable-контракт
- [ ] Тип `T` отражает реальный тип поля в context values (без лишних union)

## Связанные скиллы

- [strict-context-ref](../strict-context-ref/SKILL.md)
- [prefer-context-value-ref](../prefer-context-value-ref/SKILL.md)
- [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md)
- [instant-ui-set-context-value](../instant-ui-set-context-value/SKILL.md)
