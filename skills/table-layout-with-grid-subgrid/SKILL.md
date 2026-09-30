---
name: table-layout-with-grid-subgrid
description: >-
  Builds Matreshka BFF table-like layouts with an outer grid and nested
  subgrid rows instead of Stack or flex columns. Use when creating or reviewing
  tables, data grids, aligned rows and columns, or clickable table rows.
---

# Таблицы через Grid и Subgrid (Matreshka BFF)

## Правило

Для табличной вёрстки не собирай строки и столбцы через `stack`, `row`, `column` или другой flex-layout.

Используй:

1. внешний `grid` с единым описанием колонок;
2. вложенный `grid` для каждой логической строки;
3. `gridItem`, чтобы дочерний `grid` занял все колонки;
4. `GridTemplateColumns.Subgrid`, чтобы ячейки строки наследовали колонки таблицы.

Так заголовок и все строки используют одни и те же дорожки. Не добавляй `stack` между родительским и дочерним `grid`. Для кликабельной строки задавай `link` непосредственно дочернему `grid`.

## Шаблон

```typescript
grid(
  {
    columns: [GridColumnSize.MinContent, 1, 1],
    gap: 8,
  },
  [
    text("ID"),
    text("Имя"),
    text("Статус"),
    grid(
      {
        columns: GridTemplateColumns.Subgrid,
        gridItem: {
          columnStart: 1,
          columnEnd: -1,
        },
        link: {
          value: "/users/42",
        },
      },
      [text("42"), text("Alice"), text("Active")],
    ),
  ],
);
```

Используй `columnStart: 1` и `columnEnd: -1`, чтобы строка занимала все колонки внешнего grid независимо от их количества.

## Несколько строк

Для динамических данных возвращай дочерний `grid` непосредственно из `forEach.generator`. Строй `link.value` из `ContextRef` через `toString()`. Не добавляй контейнер вокруг grid строки и не создавай для неё независимое описание колонок: иначе `subgrid` не унаследует дорожки внешней таблицы.

```typescript
grid(
  {
    columns: [GridColumnSize.MinContent, 1, 1],
    gap: 8,
  },
  [
    text("ID"),
    text("Имя"),
    text("Статус"),
    forEach({
      ref: this.context.ref("users"),
      track: (user: UserSummary) => `${user.id}`,
      generator: ({ ref: userRef }) => [
        grid(
          {
            columns: GridTemplateColumns.Subgrid,
            gridItem: {
              columnStart: 1,
              columnEnd: -1,
            },
            link: {
              value: `./${userRef.ref("id").toString()}`,
            },
          },
          [
            text(userRef.ref("id")),
            text(userRef.ref("name")),
            text(userRef.ref("status")),
          ],
        ),
      ],
    }),
  ],
);
```

## Не делать

```typescript
// ❌ Flex описывает только одну ось и не создаёт общие дорожки таблицы
row([
  column([text("42"), text("43")]),
  column([text("Alice"), text("Bob")]),
  column([text("Active"), text("Blocked")]),
]);
```

`stack` допустим внутри отдельной ячейки, но не как прослойка между родительским grid и grid строки.

## Чеклист агента

- [ ] Табличная структура построена внешним `grid`, а не flex-контейнерами
- [ ] Колонки описаны один раз на внешнем `grid`
- [ ] Каждая логическая строка является вложенным `grid`
- [ ] Вложенный grid строки использует `GridTemplateColumns.Subgrid`
- [ ] `gridItem` дочернего grid использует `columnStart: 1` и `columnEnd: -1`
- [ ] `forEach.generator` возвращает grid строки без контейнера-прослойки
- [ ] `link` задан дочернему grid строки, а не продублирован на ячейках
- [ ] `stack` не используется для раскладки табличных колонок
