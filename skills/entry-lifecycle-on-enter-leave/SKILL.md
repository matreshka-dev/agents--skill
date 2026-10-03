---
name: entry-lifecycle-on-enter-leave
description: >-
  Uses onEnter and onLeave for Matreshka BFF entry components (Page, Dialog,
  Popover), not onShow/onHide, and avoids conditions on entry. Use when adding
  or editing Page/Dialog/Popover lifecycle handlers or enter/leave animations.
---

# Lifecycle entry-компонентов: `onEnter` / `onLeave` (Matreshka BFF)

## Правило

| Тип | Компоненты | Lifecycle-события | `conditions` |
| --- | --- | --- | --- |
| **Entry** | `Page`, `Dialog`, `Popover` | `onEnter`, `onLeave` | **нет** |
| **Regular** | `stack`, `text`, `button`, `forEach`, … | `onShow`, `onHide` | да |

- `onEnter` — entry впервые показан клиенту.
- `onLeave` — entry **начал уходить** (ещё не уничтожен окончательно).
- `onShow` / `onHide` — появление/скрытие **обычного** узла (в т.ч. из‑за `conditions`).

## Шаблон

```typescript
class FeedPage extends Page {
  protected content() {
    return [text("Лента")];
  }
}

dialog(
  {
    onEnter: animate({ duration: 200, effects: { /* … */ } }),
    onLeave: animate({ duration: 200, effects: { /* … */ } }),
  },
  [text("Содержимое")],
);
```

```typescript
// ✅ Regular: появление блока в дереве
stack(
  {
    onShow: animate({ duration: 250, effects: { /* opacity */ } }),
  },
  [text("Карточка")],
);
```

## Антипаттерны

```typescript
// ❌ Page — не onShow/onHide
class BadPage extends Page {
  protected content() {
    return [
      stack({ onShow: () => {} }, [text("…")]), // внутри дерева — ok
    ];
  }
  // ❌ не вешать onShow на сам Page как substitute for onEnter
}

// ❌ conditions на entry
new Dialog({
  conditions: [when.defined(ref)], // у entry нет conditions
  content: [text("…")],
});
```

## Чеклист

- [ ] У `Page` / `Dialog` / `Popover` анимации и lifecycle — `onEnter` / `onLeave`
- [ ] У узлов внутри дерева — `onShow` / `onHide`
- [ ] Видимость entry не через `conditions` (для regular — `conditions` допустимы)

## Связанные скиллы

- **`context-destroy-with-entry`** — уничтожение `Context` вместе с entry
- **`safe-area-for-page-dialog-overlays`** — safe area у Page/Dialog
