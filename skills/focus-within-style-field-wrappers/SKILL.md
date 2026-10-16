---
name: focus-within-style-field-wrappers
description: >-
  Uses Matreshka BFF focusWithinStyle on stack/surface/button wrappers around
  inputs and field-like controls so focus and hover apply to the outer field wrapper.
  Use when building text fields, textarea, select, date-field, or any wrapper
  where focus lives on a descendant native or input component.
---

# `focusWithinStyle` для обёрток полей (Matreshka BFF)

## Главное правило

Если **фокус клавиатуры** оказывается **на потомке** (вложенный input-компонент, textarea и т.п.), а **фон, рамка и hover/focus-цвета** должны относиться к **внешней** обёртке — включите на этой обёртке **`focusWithinStyle: true`**.

## Когда использовать

- Текстовые поля: `Surface` / `stack` вокруг `text-input`, `textarea`, `password-input`.
- «Поле-подобные» контролы с тем же визуалом поля: select / tree-select как `button` с `onClick` → popover/dialog — используйте тот же пресет обёртки, что у text-input.
- Любая **кастомная обёртка** над инпутом: border + padding + фон на stack, ввод — внутри.

## Что задать на BFF (обёртка)

| Свойство | Зачем |
| -------- | ----- |
| `focusWithinStyle: true` | Состояния hover/focus оболочки, пока фокус внутри потомков |
| `surface: true` | Фон поверхности (`Surface` / явный `surface` на stack) |
| `outline` с `OutlineType.Border` | Визуальная рамка; border берёт `ColorRole.Border` |
| `colors` | Минимум `Background`, `Border`; для содержимого — `Text`, `Placeholder`, `Icon` на обёртке или наследование — см. **`inherit-color-roles-in-tree`** |
| `radius`, `padding` | Единый «ящик» поля |

Вынесите повторяющийся набор в shared-стиль приложения (например `@styles/…` с пресетом surface + colors для полей).

Токены с состояниями **`default` / `hover` / `active`** регистрируйте в `clientSettings.colors.registry` — см. **`register-color-tokens`**.

## Когда не использовать

| Сценарий | Вместо этого |
| -------- | ------------ |
| Кликабельная кнопка, пункт меню, `link` | `onClick` или `link.value`; см. **`link-vs-navigate`** |
| Hover/active только через context + `rules` (например меню с `onMouseEnter`) | BFF conditions; `focusWithinStyle` не заменяет такую логику |
| Декоративный stack без фокусируемых потомков | `focusWithinStyle` не нужен |

## `focusWithinStyle` vs кнопка / ссылка

```text
Поле (input внутри)     →  focusWithinStyle + surface + outline + field tokens
Кнопка / ссылка         →  onClick или link, без focusWithinStyle
```

Кнопка с `onClick` без вложенного input — отдельный сценарий, не смешивать с обёрткой поля.

## Предпочтительно / избегать

```typescript
// Предпочтительно: общий пресет поля
import { fieldInputColors, fieldSurfaceStyles } from "../@styles/field";

export class ProjectTextInput extends Surface {
  constructor(config: ProjectTextInputInitConfig) {
    super({
      ...fieldSurfaceStyles,
      ...config,
      colors: {
        ...fieldSurfaceStyles.colors,
        ...fieldInputColors,
        ...config.colors,
      },
      content: config.content,
    });
  }
}
```

```typescript
// Предпочтительно: select с тем же пресетом обёртки, что у text-input
button(
  {
    ...fieldSurfaceStyles,
    onClick: () => { /* popover / dialog */ },
    colors: {
      ...fieldSurfaceStyles.colors,
      ...fieldInputColors,
    },
  },
  [text(labelRef), icon(/* chevron */)],
);
```

```typescript
// Избегать: border и фон на обёртке, но без focusWithinStyle — при фокусе в input оболочка не меняет состояние
stack(
  {
    surface: true,
    outline: [{ type: OutlineType.Border, properties: { width: 1 } }],
    colors: { [ColorRole.Background]: fieldBgToken },
  },
  [textInput({ ref: "name" })],
);

// Избегать: focusWithinStyle без surface/outline — нет визуального якоря
stack({ focusWithinStyle: true }, [textInput({ ref: "name" })]);
```

## Чеклист агента

- [ ] Фокус реально на **потомке** или планируется input внутри обёртки
- [ ] На обёртке: **`focusWithinStyle: true`**, **`surface: true`**, **`outline`** (border), **`colors`** с field-токенами
- [ ] Токены зарегистрированы (**`register-color-tokens`**), hover/active не забыты
- [ ] Не смешан с **`onClick` / `link`** там, где нужна кнопка или ссылка, а не поле
- [ ] Повторяющиеся поля переиспользуют один shared-пресет обёртки поля
