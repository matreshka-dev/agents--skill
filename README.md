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

| Скилл                                | Назначение                                                                                              |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| `context-init-in-boot`               | `await context.init()` в `boot()` до рендера, использующего `context.value()`                           |
| `context-destroy-with-entry`         | `context.destroy()` по `stopUsing$` entry (Page/Dialog), чтобы контекст не переживал страницу/диалог    |
| `client-scoped-context-for-shared-ui` | Сессионные данные chrome (badge, корзина) в одном Context на Client, не в каждом переиспользуемом виджете |
| `context-ref-in-strings`             | `ContextRef` в шаблонных строках (`toString()`, `strictRef`)                                            |
| `prefer-context-ref-in-ui`           | `ContextRef` вместо `value()` в props UI для реактивности на клиенте                                    |
| `strict-context-ref`                 | Типизация `ContextRef<T>` вместо `ContextRef<any, any>`                                                 |
| `flex-item-in-stack`                 | `flexItem` внутри flex-родителей Stack                                                                  |
| `register-color-tokens`              | Регистрация color tokens в `ClientSettings`                                                             |
| `form-submit-shared-handler`         | `form` и общий обработчик для `onSubmit` и `onClick` кнопки подтверждения                               |
| `prevent-duplicate-form-submit`      | Флаг отправки и условный `ServerAction` для защиты формы от повторного submit                           |
| `return-server-action-promise`       | Возврат или `await` Promise из серверного обработчика для передачи reject в `client.error$`             |
| `table-layout-with-grid-subgrid`     | Табличная вёрстка через общий `grid` и строки с `subgrid` вместо flex-колонок                           |
| `safe-area-with-fixed-flex-item`     | Разделение `safeArea` и фиксированного `flexItem.basis` между внешней обёрткой и внутренним контейнером |
| `safe-area-for-page-dialog-overlays` | Покрытие всех сторон safe area в `Page`, `Dialog` и их overlays без вложенного дублирования             |
| `format-after-code`                  | `npm run format` после правок в пакетах Matreshka                                                       |

## Структура

```
skills/
  <skill-name>/
    SKILL.md
```

## Лицензия

См. репозиторий matreshka-whisper / политику matreshka-dev.
