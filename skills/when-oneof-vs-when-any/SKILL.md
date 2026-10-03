---
name: when-oneof-vs-when-any
description: >-
  Picks when.oneOf for OR over one Context field versus when.any and when.all
  for compound Matreshka BFF conditions. Use when writing component or action
  conditions with AND/OR logic.
---

# `when.oneOf` vs `when.any` / `when.all` (Matreshka BFF)

## Правило

- Плоский список `conditions: [A, B, C]` — **A AND B AND C**.
- **OR по одному полю** и списку литералов → **`when.oneOf(ref, [a, b, …])`**.
- **OR между разными проверками** (разные ref, массивы, `device`) → **`when.any([…])`**.
- Явная группа AND внутри OR → **`when.all([…])`** внутри `when.any`.

## Примеры

```typescript
// Одно поле, несколько значений
when.oneOf(this.context.ref("status"), ["active", "pending"]);

// Разные поля / права
when.any([
  when.equals(this.context.ref("role"), "admin"),
  when.equals(permissions.ref("canModerate"), true),
]);

// (csv AND rows) OR (pdf AND template)
when.any([
  when.all([
    when.equals(filters.ref("format"), "csv"),
    when.notEmpty(table.ref("rows")),
  ]),
  when.all([
    when.equals(filters.ref("format"), "pdf"),
    when.defined(templates.ref("selectedId")),
  ]),
]);

// (desktop OR tablet) AND userId
conditions: [
  when.any([device.desktop(), device.tablet()]),
  when.defined(session.ref("userId")),
],
```

## Антипаттерн

```typescript
// ❌ Цепочка equals для одного ref вместо oneOf
when.equals(ref, "a"),
when.equals(ref, "b"), // это AND, не OR
```

## Чеклист

- [ ] OR одного поля → `oneOf`
- [ ] OR разных предикатов → `any` / `all`
- [ ] У action `conditions` — та же логика (см. docs events-and-actions)
