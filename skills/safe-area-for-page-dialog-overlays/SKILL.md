---
name: safe-area-for-page-dialog-overlays
description: >-
  Ensures Matreshka BFF Page and Dialog content and overlays cover all safe-area
  sides exactly once per nested branch. Use when creating or reviewing pages,
  dialogs, fullscreen content, or their overlays for notches and system UI.
---

# Safe area для Page, Dialog и overlays в Matreshka BFF

## Правило

При создании или проверке `Page` и `Dialog` учитывай устройства с вырезами
экрана и системными элементами интерфейса.

Отдельно проверь каждое дерево рендера:

1. `content` страницы;
2. каждый компонент из `Page.overlays()`;
3. `content` диалога;
4. каждый компонент из `Dialog.overlays`.

В каждом таком дереве должны быть покрыты все стороны:

- `SafeAreaSide.Top`;
- `SafeAreaSide.Bottom`;
- `SafeAreaSide.Start`;
- `SafeAreaSide.End`.

Назначай каждую сторону тому контейнеру, где это естественно для структуры
интерфейса и лаконично в коде. Контейнер не обязан быть корневым:

- `Top` обычно относится к контейнеру header;
- `Bottom` — к контейнеру нижнего меню или панели действий;
- `Start` и `End` — к контейнерам контента, header и нижней панели, которые
  доходят до соответствующих краёв экрана.

Все четыре стороны на одном корневом `stack` или его алиасе допустимы, только
если этот контейнер действительно является общей полноэкранной обёрткой и
такое решение не усложняет внутреннюю раскладку.

## Не дублируй сторону во вложенных контейнерах

Для любой ветки дерева одна сторона safe area должна применяться только один
раз. Клиент не исключает вложенные safe area автоматически: если родитель и
потомок содержат `SafeAreaSide.Top`, оба добавят верхний системный inset.

Если сторона уже назначена верхнему контейнеру, не назначай её вложенному
контейнеру. Когда вложенный компонент должен отвечать за сторону, убери эту
сторону у его предка.

## Page

```typescript
const allSafeAreaSides = [
  SafeAreaSide.Top,
  SafeAreaSide.Bottom,
  SafeAreaSide.Start,
  SafeAreaSide.End,
];

class UsersPage extends Page {
  protected content() {
    return [
      column([
        surface(
          {
            safeArea: [
              SafeAreaSide.Top,
              SafeAreaSide.Start,
              SafeAreaSide.End,
            ],
          },
          [pageHeader],
        ),
        column(
          {
            flexItem: { grow: 1 },
            safeArea: [SafeAreaSide.Start, SafeAreaSide.End],
          },
          [usersContent],
        ),
        row(
          {
            safeArea: [
              SafeAreaSide.Bottom,
              SafeAreaSide.Start,
              SafeAreaSide.End,
            ],
          },
          [bottomNavigation],
        ),
      ]),
    ];
  }

  protected overlays(): Overlay[] {
    return [
      {
        anchors: [OverlayAnchor.Bottom, OverlayAnchor.End],
        component: column(
          {
            safeArea: allSafeAreaSides,
          },
          [createUserButton],
        ),
      },
    ];
  }
}
```

Overlay страницы является отдельным деревом. Safe area в `content()` не
защищает компонент из `overlays()`, поэтому overlay проверяй независимо.

## Dialog

```typescript
new Dialog({
  content: [
    column([
      surface(
        {
          safeArea: [
            SafeAreaSide.Top,
            SafeAreaSide.Start,
            SafeAreaSide.End,
          ],
        },
        [dialogHeader],
      ),
      column(
        {
          safeArea: [SafeAreaSide.Start, SafeAreaSide.End],
        },
        [dialogContent],
      ),
      row(
        {
          safeArea: [
            SafeAreaSide.Bottom,
            SafeAreaSide.Start,
            SafeAreaSide.End,
          ],
        },
        [dialogActions],
      ),
    ]),
  ],
  overlays: [
    {
      anchors: [OverlayAnchor.Top, OverlayAnchor.End],
      component: column(
        {
          safeArea: allSafeAreaSides,
        },
        [closeButton],
      ),
    },
  ],
});
```

Для диалога `content` и каждый overlay также считаются независимыми деревьями
проверки.

## Не делать

```typescript
column(
  {
    safeArea: [SafeAreaSide.Top],
  },
  [
    column(
      {
        // ❌ Top применится второй раз
        safeArea: [SafeAreaSide.Top, SafeAreaSide.Bottom],
      },
      [content],
    ),
  ],
);
```

## Чеклист агента

- [ ] В `Page.content()` покрыты `Top`, `Bottom`, `Start` и `End`
- [ ] Каждая сторона назначена наиболее подходящему контейнеру, а не обязательно корню
- [ ] Каждый компонент из `Page.overlays()` проверен как отдельное дерево
- [ ] В `Dialog.content` покрыты все четыре стороны
- [ ] Каждый компонент из `Dialog.overlays` проверен как отдельное дерево
- [ ] Одна сторона safe area не повторяется у предка и потомка
- [ ] Используются логические стороны `Start` и `End`, а не `Left` и `Right`
- [ ] Для контейнера с фиксированным `flexItem.basis` safe area вынесена на обёртку
