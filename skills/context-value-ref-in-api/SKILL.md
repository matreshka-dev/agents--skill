---
name: context-value-ref-in-api
description: >-
  Uses Matreshka BFF ContextValueRef and ContextArrayRef in function, method,
  and shared helper signatures when only the ref value type matters—not context
  or path. Use when adding inputs/outputs helpers, platform methods, or
  refactoring ContextRef any any parameters.
---

# `ContextValueRef` в сигнатурах API (Matreshka BFF)

## Зачем

На странице ref строгий: `ContextRef<MyContext, 'amount'>` — известны контекст и путь.

В **общей функции, методе или фабрике** контекст и путь не нужны: нужен только тип **`value()`**. Параметр `ContextRef<any, any>` или даже `ContextRef<C, P>` без фиксации `C`/`P` в generic функции не даёт типизированного `value()` внутри тела.

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
  void ref.value(); // number | undefined
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

## Чеклист

- [ ] Параметры «ref на значение типа T» — **`ContextValueRef<…>`**, не `ContextRef<any, any>`
- [ ] Не требуй `ContextRef<MyPageContext, 'field'>` в shared-хелпере, если поле на странице может называться иначе
- [ ] Массивы — **`ContextArrayRef<Item>`**, не отдельный дублирующий бренд
- [ ] Generic компонента `RefType extends …ContextRef` с default **`…ContextRef`**, не `ContextRef<any, any>`

## Антипаттерн

```typescript
function bindToggle(ref: ContextRef<any, any>) {
  ref.value(); // unknown / any
}

function bindAmount(ref: ContextRef<FormContext, "amount">) {
  // жёстко привязано к одному контексту и имени поля — хелпер нельзя переиспользовать
}
```

## Связанные скиллы

- [strict-context-ref](../strict-context-ref/SKILL.md) — `ContextRef<C, P>` на странице, типы item ref в `forEach`, без `any`
- [prefer-context-ref-in-ui](../prefer-context-ref-in-ui/SKILL.md) — ref в props UI, не `value()` в дереве
