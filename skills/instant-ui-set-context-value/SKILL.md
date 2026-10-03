---
name: instant-ui-set-context-value
description: >-
  Uses Matreshka BFF LocalAction setContextValue, setContextValues, and
  toggleContextValue for immediate client UI without a BFF round-trip. Use for
  hover flags, toggles, and filters—not as the full async form submit recipe.
---

# Мгновенный UI: `setContextValue` (Matreshka BFF)

## Правило

Если UI должен обновиться **сразу на клиенте** без ожидания `server-interaction`:

- `setContextValue(ref, value)`
- `setContextValues([{ ref, value }, …])`
- `toggleContextValue(booleanRef)`

Клиент обновляет зеркало `Context` и шлёт `context-values` на BFF.

Server handler — когда нужны backend, бизнес-правила или проверка на BFF.

**Форма с loading и защитой от повторного submit** — один рецепт в **`prevent-duplicate-form-submit`** (там же `setContextValue` + `ServerAction`).

## Шаблоны

```typescript
import {
  setContextValue,
  toggleContextValue,
  setContextValues,
} from "@matreshka/bff/core";

stack(
  {
    onMouseEnter: setContextValue(this.context.ref("hovered"), true),
    onMouseLeave: setContextValue(this.context.ref("hovered"), false),
  },
  [text("Hover")],
);

button(
  { onClick: toggleContextValue(this.context.ref("enabled")) },
  [text("Переключить")],
);

button(
  {
    onClick: setContextValues([
      { ref: this.context.ref("loading"), value: true },
      { ref: this.context.ref("dirty"), value: true },
    ]),
  },
  [text("Сохранить черновик")],
);
```

## Антипаттерн

```typescript
// ❌ Только server handler для boolean loading — UI ждёт сеть
onClick: async () => {
  this.context.setValue("loading", true);
};
```

Для loading на submit используй **`setContextValue`** в массиве действий — см. **`prevent-duplicate-form-submit`**.

## Чеклист

- [ ] Hover/toggle/локальные флаги → LocalAction
- [ ] Тип `value` совпадает с типом ref
- [ ] `toggleContextValue` только для boolean ref
- [ ] Async submit + loading → **`prevent-duplicate-form-submit`**
