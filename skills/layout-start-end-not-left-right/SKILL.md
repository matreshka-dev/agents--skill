---
name: layout-start-end-not-left-right
description: >-
  Uses Matreshka BFF Start, End, OverlayAnchor, and SafeAreaSide for
  horizontal layout and RTL-safe positioning, not Left or Right. Use when
  setting padding, alignment, overlays, or safe area on stacks and pages.
---

# `Start` / `End` вместо Left / Right (Matreshka BFF)

## Правило

Горизонталь в API — **логические** стороны:

- **`Start`** / **`End`** — начало/конец строки (LTR: left/right; RTL: наоборот).
- **`OverlayAnchor.Start`**, **`OverlayAnchor.End`** — якоря overlay.
- **`SafeAreaSide.Start`**, **`SafeAreaSide.End`** — safe area.

Не использовать несуществующие или LTR-only **`Left`** / **`Right`** в props Matreshka.

## Примеры

```typescript
stack({
  padding: { start: 12, end: 12, top: 16, bottom: 16 },
  safeArea: [
    SafeAreaSide.Top,
    SafeAreaSide.Bottom,
    SafeAreaSide.Start,
    SafeAreaSide.End,
  ],
}, [text("Контент")]);
```

```typescript
{
  anchors: [OverlayAnchor.Bottom, OverlayAnchor.End],
  component: fab,
}
```

Выравнивание в row/column — `align`, `justify` с start/end там, где это предусмотрено DSL (см. docs layout).

## Чеклист

- [ ] Padding и safe area — `Start`/`End`
- [ ] Overlay anchors — не «left/right»
- [ ] Один BFF-код для LTR и RTL

## Связанные скиллы

- **`safe-area-for-page-dialog-overlays`**
- **`flex-item-in-stack`**
