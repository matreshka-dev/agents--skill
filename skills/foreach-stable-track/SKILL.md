---
name: foreach-stable-track
description: >-
  Sets a stable Matreshka BFF forEach track key from business ids, not array
  index or random values. Use when rendering lists, carts, chats, or dynamic
  rows from Context arrays.
---

# Стабильный `track` в `forEach` (Matreshka BFF)

## Правило

`track` возвращает **устойчивый id** элемента между обновлениями массива (добавление, сортировка, фильтр). Matreshka сопоставляет старые и новые строки UI.

| Хорошо | Плохо |
| --- | --- |
| `item.id`, `item.slug` | индекс, если порядок меняется |
| бизнес-ключ, не меняющийся | `Math.random()`, `Date.now()` |

## Шаблон

```typescript
forEach({
  ref: this.context.ref("items"),
  track: (item) => item.id,
  generator: ({ ref }) => text(ref.ref("title")),
});
```

Пустой список — отдельный узел с `when.isEmpty(ref)` **перед** `forEach`.

## В generator

- `ref` — ссылка на **элемент** массива;
- вложенные поля — `ref.ref("title")` (типы — **`strict-context-ref`**).

## Антипаттерн

```typescript
forEach({
  ref: this.context.ref("items"),
  track: (_item, index) => index, // ❌ при reorder UI «пересоздаётся»
  generator: ({ ref }) => text(ref.ref("title")),
});
```

## Чеклист

- [ ] У каждой сущности в массиве есть стабильный `id`
- [ ] `track` не зависит от позиции в массиве
- [ ] Один BFF-объект `forEach` в двух местах — см. docs `component-instance` (разные instance)
