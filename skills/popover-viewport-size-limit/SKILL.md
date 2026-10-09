---
name: popover-viewport-size-limit
description: >-
  Matreshka client caps Popover to the screen: with positionArea dynamic max
  to viewport edge; without positionArea CSS 100vh/100vw. Scroll on inner
  column/surface. Use when editing select-popover, Popover positionArea, or
  Overflow.Auto on popover content.
---

# Popover: лимит по экрану (Matreshka BFF)

**Не путать:** это поведение **клиента**, не поле BFF. Открытие popover — **`dialog-vs-popover-platform`**. Overlays внутри popover — **`page-overlays-method-vs-set-overlays`**.

## Правило

| Что | Как |
| --- | --- |
| Позиция рядом с якорем | `positionArea` (например `PopoverPositionArea.LogicalBlockEnd`) |
| Лимит по экрану | **Всегда** для popover из `showPopover` |
| С `positionArea` | max-height/max-width до края **visual viewport** от положения у якоря (JS, `--popover-max-*`) |
| Без `positionArea` | **max-height: 100vh**, **max-width: 100vw** (CSS); позицию задаёт браузер |
| Длинный список | `Overflow.Auto` на **внутреннем** `column` / `surface` |
| Зазор до края экрана | `padding` / `gap` у потомков в `content` |

## Шаблон (select у якоря)

```typescript
super({
  positionArea: PopoverPositionArea.LogicalBlockEnd,
  size: 1,
  content: [
    column({ padding: { vertical: 8 }, overflow: Overflow.Auto }, [
      surface({ padding: 16, overflow: Overflow.Auto /* … */ }, [
        selectOptionsList({ /* … */ }),
      ]),
    ]),
  ],
});
```

## Не делать

- Не дублировать лимит в BFF (`70vh`, лишний `basis` «чтобы влезло») — клиент уже ограничивает.
- Не ждать скролла на **корне** popover — прокрутка у потомков с `Overflow.Auto`.
- Меню **у кнопки** → задайте **`positionArea`**, не полагайтесь только на UA-позицию и 100vh/100vw.

## Связанные материалы

- Comin: `docs/matreshka/usage/components-list.md` — «Ограничение размера по экрану»

## Чеклист

- [ ] Dropdown / select у якоря → `positionArea`
- [ ] Длинный контент → `Overflow.Auto` на inner `column` или `surface`
- [ ] Отступ от края → padding/gap в `content`
