---
name: logical-px-for-bff-dimensions
description: >-
  Treats bare numeric Matreshka BFF size values (padding, gap, radius,
  flexItem basis, icon size, typography, etc.) as logical pixels for
  designers and wire payloads; client converts them to rem. Use when
  setting layout spacing, fixed sizes, or reviewing dimensional props
  without an explicit unit.
---

# Логические пиксели в размерах BFF (Matreshka)

## Главное правило

**Число без явной единицы в props размеров на BFF — это логические пиксели (px из макета), а не rem и не «сырой CSS px на клиенте».**

Клиент при рендере переводит такие значения в **`rem`** (база **16px** на единицу rem), чтобы UI масштабировался с настройками шрифта браузера и доступности.

На BFF **не нужно** вручную делить макет на 16 или писать `{ value: 1.5, unit: rem }`, если дизайнер дал **24px** — в конфиге достаточно **`24`**.

## Зачем так устроено

| На BFF | На клиенте |
| --- | --- |
| Удобно сверять с Figma/макетом (дизайнеры думают в px) | Итоговый layout не «заморожен» в физических CSS-пикселях |
| Короче wire/JSON: `padding: { top: 16 }` вместо объектов с `unit` | Единая typographic scale с root `font-size` пользователя |

Явный `{ value, unit: DimensionalUnit.* }` оставляй для **не-пиксельных** случаев (`%`, `vw`, `vh`, …) — см. skill **`flex-item-in-stack`** для `flexItem.basis`.

## Где встречаются «голые» числа

Типичные props (не исчерпывающий список — смотри JSDoc компонента):

- **`stack` / алиасы** — `gap`, `padding` (`PaddingPx`: `top`, `bottom`, `start`, `end`), `radius`
- **`flexItem.basis`** — число **`> 1`** → логические px; **`0` или `1`** → `auto` (не размер в px)
- **`icon({ size: … })`** — ширина/высота в логических px
- **`grid`** — `gap` (число или `{ row?, column? }` в px)
- **Типографика в client settings / text** — `fontSize`, `lineHeight` как числа из макета (клиент → rem)
- **Scroll, карта, анимации** — смещения и keyframes часто задаются числом или `{ value, unit: Px }` с той же семантикой

## Примеры

```typescript
// Предпочтительно: отступы и gap как в макете (px)
column(
  {
    gap: 12,
    padding: { start: 16, end: 16, top: 24, bottom: 24 },
    radius: 8,
  },
  [text("Контент")],
);
```

```typescript
import { DimensionalUnit } from "@matreshka/bff/components";

// Явная единица — когда макет не в px
stack(
  {
    flexItem: {
      basis: { value: 50, unit: DimensionalUnit.Percent },
    },
  },
  [text("Половина ширины")],
);
```

```typescript
// Предпочтительно: иконка 20×20 из спека
icon({ size: 20 }, readIcon("close"));
```

## Антипаттерны

- Писать на BFF **`rem` / `em`** там, где API ждёт **число в логических px** (или наоборот — «уменьшать» макет, деля на 16).
- Дублировать `{ value: 16, unit: DimensionalUnit.Px }` везде, где достаточно **`16`** (лишний шум в конфиге и в сообщениях клиенту).
- Путать **`flexItem.basis: 1`** (auto) с **`basis: 16`** (16 логических px).

## Чеклист агента

При правке layout и размеров в BFF:

- [ ] Значения из макета переносятся **как целые px-числа**, без ручной конвертации в rem
- [ ] Для **`flexItem.basis`** число **`> 1`** трактуется как px; для **`0`/`1`** — не «один пиксель»
- [ ] **`DimensionalValue`** с не-px unit — только когда размер **не** из макета в px
- [ ] Ожидание на клиенте: те же 16 в BFF → **`1rem`** при базе 16px, UI может масштабироваться при другом root font-size

## Связанные skills

- **`flex-item-in-stack`** — `basis`, `grow`, definite basis
- **`layout-start-end-not-left-right`** — `padding` по `start`/`end`
- **`safe-area-with-fixed-flex-item`** — px `basis`/`padding` и system insets
