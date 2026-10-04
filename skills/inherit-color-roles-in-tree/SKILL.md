---
name: inherit-color-roles-in-tree
description: >-
  Applies Matreshka BFF color roles at parent containers so children inherit
  CSS variables; avoids repeating the same ColorToken on every nested component.
  Use when styling pages, stacks, surfaces, shared layout helpers, or reviewing
  duplicated colors: { [ColorRole.*]: ... } blocks.
---

# Наследование color roles по дереву (Matreshka BFF)

## Главное правило

**Роли цвета каскадируют по ветке.** Клиент выставляет переменные роли на узле `ServerComponentWrapper`; потомки **наследуют** их, пока своя ветка не задаст `colors` заново.

Не дублируйте одни и те же `ColorToken` на каждом вложенном `text()`, `icon()` или `button()`, если родитель уже задал нужные роли для блока.

Регистрация токенов в `clientSettings.colors.registry` по-прежнему обязательна — см. skill **`register-color-tokens`**.

## Откуда берётся палитра

| Уровень            | Где задаётся                                                         | Кто наследует                                                        |
| ------------------ | -------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Приложение         | `clientSettings.colors.default` на `html`                            | Overlays платформы, узлы **без** своих `colors`, subtree под wrapper |
| Секция / контейнер | `colors` на `stack`, `surface`, `column`, `row`, Page, Dialog и т.д. | Вся вложенная ветка, пока дочерний узел не переопределит роли        |
| Локально           | `colors` на отдельном компоненте                                     | Только эта ветка; перекрывает предка и `default` для своих потомков  |

Локальные `properties.colors` **не включают** border, shadow и прочий chrome сами по себе — компонент должен использовать роль в своём рендере. Токен только задаёт цвет для роли, которую компонент уже рисует.

## Когда задавать `colors` на родителе

- Корень страницы, диалога или крупный блок (карточка, панель, список) — **одна палитра на секцию**.
- Несколько ролей сразу (`Background`, `Text`, `Icon`, `Border`, …), если внутри будут `text`, `icon`, bordered-компоненты и кнопки, берущие эти роли.

## Когда задавать `colors` на потомке

- Ветка **должна отличаться** от родителя (accent-кнопка на нейтральном фоне, вторичный текст, отдельная «островная» surface).
- Роль нужна только здесь и **не задана** предком (например `ColorRole.Link` у `text()` с link-процессором).
- Переиспользуемый компонент **сам** задаёт палитру и не должен полагаться на неизвестного предка (библиотечный виджет с фиксированным контрастом).

## Антипаттерны

- Копировать один и тот же блок `colors: { [ColorRole.Background]: …, [ColorRole.Text]: … }` на каждый дочерний элемент.
- Повторять `Text` / `Icon` на листьях, когда родительский `stack`/`surface` уже задаёт эти роли для секции.
- Ожидать, что `ColorRole.Border` нарисует границу без компонента, который border реально рисует.

## Предпочтительно / избегать

```typescript
// Предпочтительно: палитра секции на контейнере
stack(
  {
    colors: {
      [ColorRole.Background]: surfaceColorToken,
      [ColorRole.Text]: primaryTextColorToken,
      [ColorRole.Icon]: accentIconColorToken,
    },
  },
  [
    text("Заголовок"),
    text("Описание"),
    button([icon("edit"), text("Изменить")]),
  ],
);

// Избегать: те же токены на каждом ребёнке
stack({}, [
  text({ colors: { [ColorRole.Text]: primaryTextColorToken } }, "Заголовок"),
  text({ colors: { [ColorRole.Text]: primaryTextColorToken } }, "Описание"),
]);
```

```typescript
// Предпочтительно: локальное переопределение только где нужен акцент
stack(
  {
    colors: {
      [ColorRole.Background]: bgPrimaryColorToken,
      [ColorRole.Text]: textPrimaryColorToken,
    },
  },
  [
    text("Обычный текст"),
    button(
      {
        colors: {
          [ColorRole.Background]: bgAccentColorToken,
          [ColorRole.Text]: textOnAccentColorToken,
        },
      },
      [text("Действие")],
    ),
  ],
);
```

## Чеклист агента

При правке цветов в конфиге BFF:

- [ ] Общая палитра секции задана **один раз** на контейнере или в `colors.default`, если это глобальная тема
- [ ] На листьях `colors` только там, где палитра **отличается** от предка или нужна отдельная роль
- [ ] Новые токены из `colors` по-прежнему в **`clientSettings.colors.registry`**
- [ ] Не дублируются идентичные назначения ролей без причины (рефакторинг в родителя)
