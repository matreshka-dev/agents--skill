---
name: page-overlays-method-vs-set-overlays
description: >-
  Chooses Matreshka Page overlays() versus currentClientPlatform setOverlays
  in boot for FAB and snackbars—not showDialog/showPopover or stack.overlays.
---

# Overlays страницы: `overlays()` vs `setOverlays()` (Matreshka BFF)

**Не путать:** это **наложение компонента на страницу/клиента** (якоря `OverlayAnchor`). Модальные **Dialog/Popover** через **`showDialog` / `showPopover`** — скилл **`dialog-vs-popover-platform`**. Локальный badge на **`stack.overlays`** — docs overlays.md.

## Правило

| Способ | Когда |
| --- | --- |
| **`protected overlays()`** у `Page` | FAB, нижняя панель, snackbar **на уровне страницы** — декларативно в классе Page |
| **`currentClientPlatform().setOverlays([…])`** | Глобальные overlays клиента; часто в **`boot()`** |

Якоря — **`OverlayAnchor.Start` / `End`** (скилл **`layout-start-end-not-left-right`**).

## Шаблон Page

```typescript
class ListPage extends Page {
  protected overlays() {
    return [
      {
        anchors: [OverlayAnchor.Bottom, OverlayAnchor.Center],
        component: button([text("Создать")]),
      },
    ];
  }

  protected content() {
    return [text("Список")];
  }
}
```

## Шаблон `setOverlays` в boot

```typescript
override async boot() {
  currentClientPlatform().setOverlays([
    {
      anchors: [OverlayAnchor.Bottom, OverlayAnchor.Center],
      component: button([text("Создать")]),
    },
  ]);

  return super.boot();
}
```

Согласуй порядок с **`await super.boot()`** и **`context-init-in-boot`**, если overlays зависят от `value()`.

## Чеклист

- [ ] FAB/snackbar на маршруте → `Page.overlays()` или `setOverlays`
- [ ] Подтверждение/меню по клику → **`dialog-vs-popover-platform`**
- [ ] Safe area → **`safe-area-for-page-dialog-overlays`**
