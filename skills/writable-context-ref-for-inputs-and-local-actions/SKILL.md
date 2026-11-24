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

`WritableContextRefOfValue<T>` не требует generic по ref (`R`): writable-принадлежность выражена структурно (`local.set` без `{ sync: false }` есть только у writable-путей), а `local` выводится из `ContextRef.local`. Ref с путём вне схемы не пройдёт проверку вызова без флага. Новые методы `local` подхватываются автоматически.

## Когда какой тип использовать

| Сценарий                                                                | Тип                            |
| ----------------------------------------------------------------------- | ------------------------------ |
| Нужен ref по типу значения, запись не обязательна                       | `ContextRefOfValue<T>`         |
| Нужен ref, который синхронизируется с BFF (input, `ref.local.*` без `sync: false`) | `WritableContextRefOfValue<T>` |

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

## Local-действия Context

`ref.local.set(...)`, `ref.local.toggle()`, `ref.local.inc(...)`, `ref.local.dec(...)` без `{ sync: false }` вызываются на writable ref: путь есть в схеме и уходит на BFF.

`{ sync: false }` пишет только клиентское зеркало. Для этого вызова хватает ref по типу значения, путь в схеме не нужен. `WritableContextRefOfValue` по-прежнему означает, что `local.*` можно вызвать без этого флага. Когда ставить `sync: false` — **`context-local-sync-false`**.

```typescript
// Client-only loading lock: local с sync: false; сброс на BFF — ref.set
function lockSubmit(loadingRef: ContextRefOfValue<boolean | undefined>) {
  return loadingRef.local.set(true, { sync: false });
}

function unlockSubmit(loadingRef: ContextRefOfValue<boolean | undefined>) {
  return loadingRef.set(false);
}

function hover(hoveredRef: ContextRefOfValue<boolean | undefined>) {
  return hoveredRef.local.set(true, { sync: false });
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
- [context-local-sync-false](../context-local-sync-false/SKILL.md)
