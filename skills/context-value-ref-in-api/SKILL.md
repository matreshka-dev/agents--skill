---
name: context-value-ref-in-api
description: >-
  Uses Matreshka BFF ContextValueRef and ContextArrayRef in function, method,
  and shared helper signatures when only the ref value type matters—not context
  or path. Prefer ref.get() and ref.set() over ref.context when a ref is
  already in hand. Prefer ref.set() over duplicating paths with context.set.
  Use when adding inputs/outputs helpers, platform methods,
  ServerAction finally blocks, or refactoring ContextRef any any parameters.
---

# `ContextValueRef` в сигнатурах API (Matreshka BFF)

## Зачем

На странице ref строгий: `ContextRef<MyContext, 'amount'>` — известны контекст и путь.

В **общей функции, методе или фабрике** контекст и путь не нужны: нужен только тип **`get()`**. Параметр `ContextRef<any, any>` или даже `ContextRef<C, P>` без фиксации `C`/`P` в generic функции не даёт типизированного `get()` внутри тела.

Используй **`ContextValueRef<ValueType>`** — call site по-прежнему передаёт `this.context.ref('…')`; TypeScript проверяет совместимость типа значения.

## Когда какой тип

| Ситуация                                                        | Тип параметра                                                                      |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Хелпер / метод принимает ref на число, строку, boolean          | `ContextValueRef<T \| undefined>` или alias (`NumberInputContextRef`)              |
| Хелпер принимает ref на массив для списка / `forEach`           | `ContextArrayRef<ItemType>`                                                        |
| Конфиг компонента на call site (сохранить конкретный `R`)       | `CompatibleContextValueRef<V, R>` или `CompatibleInputRef` / `CompatibleOutputRef` |
| Нужны вложенные `ref('a.b')` **от параметра** с проверкой путей | Оставь `ContextRef<C, P>` с constraint на `C` (скилл **strict-context-ref**)       |

## Пример

```typescript
import { ContextValueRef } from "@matreshka/bff/core";
import { numberInput } from "@matreshka/bff/components/inputs/number-input";

type AmountRef = ContextValueRef<number | undefined>;

function amountField(ref: AmountRef, placeholder: string) {
  void ref.get(); // number | undefined
  return numberInput({ placeholder }, ref);
}

// На странице:
amountField(this.context.ref("amount"), "Сумма"); // OK
// amountField(this.context.ref('title'), '…');   // ошибка TS
```

Списки:

```typescript
import { ContextArrayRef } from '@matreshka/bff/core';

function historyBlock(ref: ContextArrayRef<ChatMessage>) {
  return forEach({ ref, track: (m) => m.id, generator: … });
}
```

## Чтение и запись через ref

Если **ref уже есть** (параметр хелпера, `const loadingRef = …`, ref из `forEach`), читай и пиши **через ref**, не разбирай `ref.context` и не дублируй путь строкой.

| Задача | Предпочтительно | Когда иначе |
| ------ | --------------- | ----------- |
| Прочитать значение | `ref.get()` | Поля ещё нет в переменной — `this.context.get("path")` |
| Записать на BFF (handler, `ServerAction`, сервис) | `ref.set(value)` | Ref ещё не создан — `this.context.set({ path: value })` |
| Мгновенно на клиенте в **массиве actions** | `setContextValue(ref, value)` | Не заменяет `ref.set` в async-callback на BFF — см. [instant-ui-set-context-value](../instant-ui-set-context-value/SKILL.md) |

`ref.set()` на BFF вызывает `context.set({ … })` по пути ref и синхронизирует клиент. Если контекст уже уничтожен, вызов **игнорируется** — удобно в `finally` после `await`.

```typescript
import type { ContextValueRef } from "@matreshka/bff/core";

function resetLoading(loadingRef: ContextValueRef<boolean | undefined>) {
  loadingRef.set(false);
}

// На странице после async submit:
const loadingRef = this.context.ref("loading");
// …
finally {
  loadingRef.set(false);
}
```

## Чеклист

- [ ] Параметры «ref на значение типа T» — **`ContextValueRef<…>`**, не `ContextRef<any, any>`
- [ ] Не требуй `ContextRef<MyPageContext, 'field'>` в shared-хелпере, если поле на странице может называться иначе
- [ ] Массивы — **`ContextArrayRef<Item>`**, не отдельный дублирующий бренд
- [ ] Generic компонента `RefType extends …ContextRef` с default **`…ContextRef`**, не `ContextRef<any, any>`
- [ ] При наличии ref — **`ref.get()` / `ref.set()`**, не `ref.context.set(…)` с кастами
- [ ] Не пиши `this.context.set({ field: … })`, если в scope уже есть ref на то же поле

## Антипаттерн

```typescript
function bindToggle(ref: ContextRef<any, any>) {
  ref.get(); // unknown / any
}

function bindAmount(ref: ContextRef<FormContext, "amount">) {
  // жёстко привязано к одному контексту и имени поля — хелпер нельзя переиспользовать
}

function resetLoading(loadingRef: ContextValueRef<boolean | undefined>) {
  loadingRef.set(false);
}
```

## Связанные скиллы

- [strict-context-ref](../strict-context-ref/SKILL.md) — `ContextRef<C, P>` на странице, типы item ref в `forEach`, без `any`
- [prefer-context-ref-in-ui](../prefer-context-ref-in-ui/SKILL.md) — ref в props UI, не `get()` в дереве
- [instant-ui-set-context-value](../instant-ui-set-context-value/SKILL.md) — `setContextValue` в actions
- [prevent-duplicate-form-submit](../prevent-duplicate-form-submit/SKILL.md) — сброс `loadingRef` после submit
