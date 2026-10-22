---
name: prefer-context-get-set-over-replace-snapshot
description: >-
  Prefers Matreshka BFF Context get and set(patch) over snapshot and replace
  for reads and partial writes. Use when editing Page/Dialog handlers, reload
  helpers, or reviewing context state updates in matreshka config.
---

# `get` / `set` вместо `snapshot` / `replace` (Matreshka BFF)

## Правило

Для чтения и записи состояния контекста на BFF **по умолчанию** используй точечные API:

| Задача | Предпочтительно | Реже |
|--------|-----------------|------|
| Прочитать поле | `context.get("path")`, `ref.get()` | `snapshot()` ради одного поля |
| Обновить одно или несколько полей | `context.set({ … })`, `ref.set(value)` | `replace()` |
| Собрать объект для UI/логики | несколько `get` / уже имеющиеся refs | `snapshot()` «на всякий случай» |

`replace(data)` и `snapshot()` — для **целого снимка** корня `T`, не для обычной правки формы, таба или чата.

## `set` (patch)

Один вызов — несколько dotted-путей, один цикл синхронизации с клиентом:

```typescript
const history = this.context.get("history");

this.context.set({
  message: "",
  sending: false,
  history: [...history, item],
});
```

Смена таба и производного списка:

```typescript
this.context.set({
  activeCategoryTabValue: tab,
  documents: filtered.slice(0, this.documentsLoadedCount),
});
```

Явный `undefined` в patch — намеренная запись (не «пропустить ключ»).

## `get`

```typescript
const tab = this.context.get("activeCategoryTabValue");
const draft = this.context.get("message");
```

Если в scope уже есть `ContextRef` на то же поле — **`ref.get()` / `ref.set(...)`** (см. [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md)).

## Когда уместны `replace` и `snapshot`

**`replace(data: T)`** — новый корневой снимок целиком:

- результат `await load…()` без точечных правок;
- merge на сервере уже собран: `replace({ ...loaded, message: preserved })`.

**`snapshot(): T`** — нужна **отсоединённая копия всего** `T` (structured clone), например перед долгим `await`, чтобы не опираться на живое состояние, которое могло измениться. Для одного-двух полей — `get`, не `snapshot`.

## Антипаттерны

```typescript
// ❌ replace ради двух полей
const data = this.context.snapshot();
this.context.replace({ ...data, sending: false, message: "" });

// ✅
this.context.set({ sending: false, message: "" });
```

```typescript
// ❌ snapshot, чтобы прочитать tab
const tab = this.context.snapshot().activeCategoryTabValue;

// ✅
const tab = this.context.get("activeCategoryTabValue");
```

Не мутируйте результат `get` «на месте» — соберите новое значение и вызовите `set` или `replace`.

## Чеклист

- [ ] Чтение — `get` / `ref.get()`, не `snapshot` без причины
- [ ] Запись — `set({ … })` / `ref.set`, не `replace` для частичных правок
- [ ] `replace` только для полной подмены или готового снимка после load/merge
- [ ] `snapshot` только когда нужна копия **всего** состояния на время async
