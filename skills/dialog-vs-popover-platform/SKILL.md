---
name: dialog-vs-popover-platform
description: >-
  Chooses Matreshka showDialog versus showPopover (source or { anchor, source })
  for confirmations and contextual menus. Use when opening entry layers from
  onClick—not for Page.overlays() or stack overlays.
---

# `showDialog` vs `showPopover` (Matreshka BFF)

**Не путать:** это **entry-слои** через platform API. Декларативные **`Page.overlays()`** / **`stack.overlays`** — скилл **`page-overlays-method-vs-set-overlays`**. Safe area у Dialog — **`safe-area-for-page-dialog-overlays`**.

## Правило

| Сценарий                                                        | API                                         | Кого передать                          |
| --------------------------------------------------------------- | ------------------------------------------- | -------------------------------------- |
| Подтверждение, отдельный ввод, слой **без** привязки к элементу | `showDialog(new Dialog({ … }))`             | —                                      |
| Меню/действия **рядом с** кнопкой, карточкой, строкой           | `showPopover(instance, new Popover({ … }))` | **source** = `instance` из события     |
| Открыли с одного элемента, показываем **у другого** (шапка, поле) | `showPopover({ source, anchor }, popover)`  | **source** из `onClick`, **anchor** — заранее известный инстанс |

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
          content: [button([text("Редактировать")]), button([text("Удалить")])],
        }),
      );
    },
  },
  [text("Действия")],
);
```

```typescript
// Клик по кнопке в строке, popover у шапки (headerInstance — createInstance или ref из boot)
button(
  {
    onClick: ({ instance }) => {
      currentClientPlatform().showPopover(
        { source: instance, anchor: headerInstance },
        new Popover({
          positionArea: PopoverPositionArea.LogicalBlockEnd,
          content: [/* … */],
        }),
      );
    },
  },
  [text("Фильтр")],
);
```

## Связанные скиллы

- **`popover-viewport-size-limit`** — лимит по экрану, `positionArea`, скролл у потомков
- **`entry-lifecycle-on-enter-leave`** — `Dialog` / `Popover` как entry
- **`context-destroy-with-entry`** — Context внутри dialog
- **`create-instance-before-serialize`** — `instance` якоря popover

## Чеклист

- [ ] «Рядом с этой кнопкой» → `showPopover(instance, …)` (один source)
- [ ] «Открыл здесь, висит у другого блока» → `{ source: instance, anchor: … }`
- [ ] Модальный сценарий без якоря → dialog
- [ ] FAB/snackbar на странице → не showDialog, а **`page-overlays-method-vs-set-overlays`**
