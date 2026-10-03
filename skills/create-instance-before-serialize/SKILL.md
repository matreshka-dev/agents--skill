---
name: create-instance-before-serialize
description: >-
  Uses Matreshka BFF Component createInstance for animate componentId, comparing
  event instance, or targeting one DOM instance when one Component appears twice.
  Use when reusing one button or text node in multiple tree places.
---

# `createInstance()` и instance id (Matreshka BFF)

## Правило

- Один BFF-объект `Component` может сериализоваться **несколько раз** → instance id `{baseId}-0`, `{baseId}-1`, …
- События приходят с **`instance`** того DOM-узла, где клик/hover.
- **`createInstance()`** — когда на BFF **заранее** нужна ссылка на конкретный будущий инстанс.

Для простого дублирования в дереве достаточно два раза положить один `Component` — инстансы создадутся при serialize.

## Когда нужен явный instance

**Анимация другого компонента:**

```typescript
const banner = text("Сохранено");

[
  banner,
  button(
    {
      onClick: animate({
        componentId: banner.id,
        duration: 300,
        effects: { /* opacity */ },
      }),
    },
    [text("Подсветить")],
  ),
];
```

**Различить две кнопки с общим handler:**

```typescript
const demoButton = button(
  {
    onClick: ({ instance }) => {
      if (instance === firstButton) { /* … */ }
      if (instance === secondButton) { /* … */ }
    },
  },
  [text("Текст")],
);

const firstButton = demoButton.createInstance();
const secondButton = demoButton.createInstance();
```

**Popover:** anchor — `instance` из `onClick`, не базовый `component.id`.

## Ограничение

`instance.serialize()` вызывается **один раз** на instance; повтор — ошибка. Обновление UI — команды BFF→client или новый instance.

## Чеклист

- [ ] `componentId` в `animate` — id нужного **инстанса** или базовый id компонента по docs
- [ ] `showPopover(instance, …)` — instance из события
- [ ] Не путать базовый id Component и instance id в interaction `target`

## Связанные скиллы

- **`dialog-vs-popover-platform`**
