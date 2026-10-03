---
name: dialog-vs-popover-platform
description: >-
  Chooses Matreshka currentClientPlatform showDialog versus showPopover with
  anchor instance for confirmations and contextual menus. Use when opening
  entry layers from onClick—not for Page.overlays() or stack overlays.
---

# `showDialog` vs `showPopover` (Matreshka BFF)

**Не путать:** это **entry-слои** через platform API. Декларативные **`Page.overlays()`** / **`stack.overlays`** — скилл **`page-overlays-method-vs-set-overlays`**. Safe area у Dialog — **`safe-area-for-page-dialog-overlays`**.

## Правило

| Сценарий | API | Якорь |
| --- | --- | --- |
| Подтверждение, отдельный ввод, слой **без** привязки к элементу | `showDialog(new Dialog({ … }))` | нет |
| Меню/действия **рядом с** кнопкой, карточкой, строкой | `showPopover(instance, new Popover({ … }))` | **instance** из события |

`Popover` всегда привязан к anchor-компоненту на клиенте.

## Шаблоны

```typescript
button(
  {
    onClick: () => {
      currentClientPlatform().showDialog(
        new Dialog({ content: [text("Вы уверены?")] }),
      );
    },
  },
  [text("Удалить")],
);
```

```typescript
button(
  {
    onClick: ({ instance }) => {
      currentClientPlatform().showPopover(
        instance,
        new Popover({
          content: [
            button([text("Редактировать")]),
            button([text("Удалить")]),
          ],
        }),
      );
    },
  },
  [text("Действия")],
);
```

## Связанные скиллы

- **`entry-lifecycle-on-enter-leave`** — `Dialog` / `Popover` как entry
- **`context-destroy-with-entry`** — Context внутри dialog
- **`create-instance-before-serialize`** — `instance` якоря popover

## Чеклист

- [ ] «Рядом с этой кнопкой» → popover + `instance`
- [ ] Модальный сценарий без якоря → dialog
- [ ] FAB/snackbar на странице → не showDialog, а **`page-overlays-method-vs-set-overlays`**
