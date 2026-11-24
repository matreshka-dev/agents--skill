---
name: instant-ui-set-context-value
description: >-
  Uses Matreshka BFF context.local.set, ref.local.set, context.local.toggle,
  context.local.inc, and context.local.dec for immediate client UI without a
  server-interaction round-trip. By default the client also syncs via
  context-values; use { sync: false } for client-only flags (see
  context-local-sync-false). Use for hover flags, toggles, counters, and
  filters—not as the full async form submit recipe.
---

# Мгновенный UI: `context.local` / `ref.local` (Matreshka BFF)

## Правило

Если UI должен обновиться **сразу на клиенте** без ожидания `server-interaction`:

- `context.local.set(patch, config?)` — как `context.set`, patch по dotted-путям
- `ref.local.set(value, config?)` — одно поле по уже имеющемуся ref
- `context.local.toggle(path, config?)` — только boolean-пути в типе контекста
- `ref.local.toggle(config?)` — инверсия по ref, если `PathValue` — boolean
- `context.local.inc(path, step?, config?)` — только числовые пути; клиент прибавляет `step` (по умолчанию `1`)
- `ref.local.inc(step?, config?)` — то же по уже имеющемуся ref, если `PathValue` — number
- `context.local.dec(path, step?, config?)` — числовые пути; клиент вычитает `step` (по умолчанию `1`)
- `ref.local.dec(step?, config?)` — то же по ref с числовым `PathValue`

По умолчанию клиент обновляет зеркало `Context` и шлёт `context-values` на BFF
(путь в `schema`). Если серверу значение не нужно — `{ sync: false }`: только
зеркало, ключ в schema не клади. Критерий и примеры — **`context-local-sync-false`**.

Server handler — когда нужны backend, бизнес-правила или проверка на BFF.

**Форма с loading и защитой от повторного submit** — **`prevent-duplicate-form-submit`**.

## Шаблоны

```typescript
import { local } from "@matreshka/bff/core";

stack(
  {
    onMouseEnter: this.context.local.set({ hovered: true }, { sync: false }),
    onMouseLeave: this.context.local.set({ hovered: false }, { sync: false }),
  },
  [text("Hover")],
);

button(
  { onClick: this.context.local.toggle("enabled") },
  [text("Переключить")],
);

button(
  { onClick: enabledRef.local.toggle() },
  [text("Переключить")],
);

button(
  { onClick: this.context.local.inc("quantity") },
  [text("Добавить")],
);

button(
  { onClick: this.context.local.dec("quantity") },
  [text("Убрать")],
);

button(
  { onClick: quantityRef.local.dec() },
  [text("Убрать")],
);

button(
  {
    onClick: this.context.local.set({ dirty: true }),
  },
  [text("Сохранить черновик")],
);

button(
  {
    onClick: loadingRef.local.set(true, { sync: false }),
  },
  [text("Submit")],
);

button(
  {
    onClick: deviceRef.local.set("off", {
      conditions: [when.equals(deviceRef.ref("status"), "on")],
    }),
  },
  [text("Power")],
);
```

Глобальные client-only действия: `local.animate(...)`, `local.writeTextToClipboard(text, config?)`.

Entry dialog/popover: **`this.local.close()`** — закрытие без round-trip на BFF (в payload `target` — instance id entry при сериализации дерева). Серверное **`this.close()`** — когда нужен ack с BFF.

## Антипаттерн

```typescript
// ❌ Только server handler для boolean loading — UI ждёт сеть
onClick: async () => {
  this.context.set({ loading: true });
};
```

Для loading на submit — **`ref.local.set(true, { sync: false })`** — см. **`prevent-duplicate-form-submit`** и **`context-local-sync-false`**.

## Чеклист

- [ ] Hover/toggle/локальные флаги → LocalAction через `.local`
- [ ] Client-only флаги (hover, form loading) → `{ sync: false }` — **`context-local-sync-false`**
- [ ] Patch / value совпадают с типом контекста
- [ ] Toggle через `context.local.toggle(path)` или `ref.local.toggle()`
- [ ] Inc/dec через `context.local.inc|dec(path, step?)` или `ref.local.inc|dec(step?)` для number
- [ ] Async submit + loading → **`prevent-duplicate-form-submit`**
