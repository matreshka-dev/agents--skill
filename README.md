# Matreshka Agent Skills

Набор [Agent Skills](https://agentskills.io/) для работы с Matreshka BFF в Cursor и других AI-агентах.

## Установка

Из GitHub:

```bash
npx skills add matreshka-dev/agents--skill
```

Только в Cursor:

```bash
npx skills add matreshka-dev/agents--skill --agent cursor
```

Конкретные скиллы:

```bash
npx skills add matreshka-dev/agents--skill --skill context-init-in-boot prefer-context-ref-in-ui
```

Локально (для разработки пакета):

```bash
npx skills add . --agent cursor
```

## Скиллы

### Context и состояние

| Скилл                                  | Назначение                                                   |
| -------------------------------------- | ------------------------------------------------------------ |
| `context-init-in-boot`                 | Preload / lazy / `init()` в `boot()` при `value()` в рендере |
| `context-plan-destroy-on-create`       | План `destroy()`; default auto при last client; `persistent: true` и ручная инвалидация |
| `context-destroy-with-entry`           | `context.destroy()` по `stopUsing$` entry (Page/Dialog)      |
| `client-scoped-context-for-shared-ui`  | Один Context на Client для chrome (badge, корзина)           |
| `context-ref-in-strings`               | `ContextRef` в шаблонных строках (`toString()`, `strictRef`) |
| `prefer-context-ref-in-ui`             | `ContextRef` в props UI вместо `value()`                     |
| `strict-context-ref`                   | `ContextRef<C, P>` на странице, без `any`                    |
| `context-value-ref-in-api`             | `ContextValueRef` / `ContextArrayRef` в сигнатурах хелперов  |
| `storage-vs-context`                   | Клиентский `storage` vs состояние сценария в `Context`       |
| `context-subscribe-take-until-destroy` | `data$` с `takeUntil(destroy$)` на entry                     |
| `context-zod-draft-not-live-strict`    | Zod: draft vs strict на живом вводе                          |
| `emit-after-in-place-mutation`         | `emitAfterInPlaceMutation()` после in-place мутации          |

### Компоненты, условия, списки

| Скилл                                 | Назначение                                                      |
| ------------------------------------- | --------------------------------------------------------------- |
| `entry-lifecycle-on-enter-leave`      | `onEnter`/`onLeave` у Page/Dialog/Popover; не `onShow` на entry |
| `rules-not-conditions-for-loading-ui` | Loading-текст через `rules`, не скрытие кнопки                  |
| `foreach-stable-track`                | Стабильный `track` в `forEach` (id, не index)                   |
| `when-oneof-vs-when-any`              | `oneOf` vs `any`/`all` в conditions                             |
| `create-instance-before-serialize`    | `createInstance()`, `componentId`, `instance` в событиях        |
| `register-color-tokens`               | Color tokens в `ClientSettings`                                 |

### События, формы, навигация

| Скилл                               | Назначение                                                   |
| ----------------------------------- | ------------------------------------------------------------ |
| `form-submit-shared-handler`        | Общий handler для `onSubmit` и `onClick`                     |
| `prevent-duplicate-form-submit`     | `loading` + `setContextValue` + conditional `ServerAction`   |
| `return-server-action-promise`      | Return/await Promise в server handlers                       |
| `server-action-validate-in-handler` | Бизнес-проверки в handler, не только `conditions` на клиенте |
| `instant-ui-set-context-value`      | `setContextValue` / toggle для мгновенного UI                |
| `link-vs-navigate`                  | `link.value` vs `navigate()` в `onClick`                     |
| `route-guard-return-undefined`      | Guard через `undefined` и fallback `**`                      |
| `dialog-vs-popover-platform`        | `showDialog` vs `showPopover(instance, …)`                   |
| `popover-viewport-size-limit`       | Лимит popover по экрану; `positionArea` vs 100vh/100vw       |

### Layout и overlays

| Скилл                                  | Назначение                                              |
| -------------------------------------- | ------------------------------------------------------- |
| `flex-item-in-stack`                   | `flexItem` внутри flex-родителей Stack                  |
| `logical-px-for-bff-dimensions`        | Числа размеров на BFF = логические px; на клиенте → rem |
| `layout-start-end-not-left-right`      | `Start`/`End`, RTL-safe якоря и safe area               |
| `safe-area-with-fixed-flex-item`       | Safe area и фиксированный `flexItem.basis`              |
| `safe-area-for-page-dialog-overlays`   | Safe area в Page/Dialog/overlays                        |
| `table-layout-with-grid-subgrid`       | Таблица через `grid` + `subgrid`                        |
| `page-overlays-method-vs-set-overlays` | `Page.overlays()` vs `setOverlays()` в `boot()`         |

## Структура

```
skills/
  <skill-name>/
    SKILL.md
```

## Лицензия

См. репозиторий matreshka-whisper / политику matreshka-dev.
