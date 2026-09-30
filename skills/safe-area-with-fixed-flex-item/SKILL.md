---
name: safe-area-with-fixed-flex-item
description: >-
  Separates Matreshka BFF safeArea from a fixed pixel flexItem basis so system
  insets do not shrink content. Use when building fixed-height bottom or top
  bars, fixed-width side panels, fullscreen layouts, dialogs, or overlays.
---

# `safeArea` с фиксированным `flexItem` в Matreshka BFF

## Правило

Не задавай пиксельный `flexItem.basis` и `safeArea` одному контейнеру.

Создай два уровня:

1. внешний Stack или его алиас (`column`, `row`, `surface`) содержит только `safeArea`;
2. внутренний контейнер содержит фиксированный `flexItem` и нужный `padding`.

У обёртки с `safeArea` не должно быть фиксированного `flexItem` или `padding`.

## Почему

Пиксельный `flexItem.basis` задаёт размер контейнера целиком, включая `padding`.
Safe area прибавляется к padding, поэтому на одном контейнере системный inset
уменьшает доступную контентную область:

```text
контент = basis − padding по оси − safe area по оси
```

Значение inset определяется клиентом и платформой, поэтому его нельзя заранее
учесть в `basis` на BFF.

## Правильный паттерн

```typescript
column(
  {
    safeArea: [SafeAreaSide.Bottom],
  },
  [
    surface(
      {
        direction: StackDirection.Horizontal,
        flexItem: { basis: 56 },
        padding: { horizontal: 16 },
      },
      [text("Нижняя панель")],
    ),
  ],
);
```

Внешний контейнер увеличивается на системный inset. Внутренний сохраняет
заданный `basis`, а его контент не сжимается из-за safe area.

## Учитывай ось родителя

`flexItem` действует по главной оси родительского Stack:

- в `column` фиксируется высота, поэтому конфликтуют `Top` и `Bottom`;
- в `row` фиксируется ширина, поэтому конфликтуют `Start` и `End`.

При `flexItem: { grow: 1 }` контейнер занимает долю свободного места, поэтому
`padding` и `safeArea` не отнимают фиксированный пиксельный бюджет.

## Не делать

```typescript
// ❌ Системный inset уменьшит место для контента внутри basis: 56
surface(
  {
    flexItem: { basis: 56 },
    padding: { horizontal: 16 },
    safeArea: [SafeAreaSide.Bottom],
  },
  [text("Нижняя панель")],
);
```

## Чеклист агента

- [ ] `safeArea` находится на внешнем Stack или его алиасе
- [ ] У обёртки с `safeArea` нет фиксированного `flexItem` и `padding`
- [ ] Фиксированный `flexItem.basis` находится на внутреннем контейнере
- [ ] `padding` находится на внутреннем контейнере
- [ ] Сторона safe area проверена относительно главной оси родителя
