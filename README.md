# Matreshka Agent Skills

Набор [Agent Skills](https://agentskills.io/) для работы с Matreshka BFF в Cursor и других AI-агентах.

Скиллы перенесены из репозитория [matreshka-whisper](https://github.com/matreshka-dev/matreshka-whisper) (`.cursor/skills/`).

## Установка

Из GitHub (после публикации репозитория):

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

| Скилл | Назначение |
| --- | --- |
| `context-init-in-boot` | `await context.init()` в `boot()` до рендера, использующего `context.value()` |
| `context-ref-in-strings` | `ContextRef` в шаблонных строках (`toString()`, `strictRef`) |
| `prefer-context-ref-in-ui` | `ContextRef` вместо `value()` в props UI для реактивности на клиенте |
| `strict-context-ref` | Типизация `ContextRef<T>` вместо `ContextRef<any, any>` |
| `flex-item-in-stack` | `flexItem` внутри flex-родителей Stack |
| `register-color-tokens` | Регистрация color tokens в `ClientSettings` |
| `format-after-code` | `npm run format` после правок в пакетах Matreshka |

## Структура

```
skills/
  <skill-name>/
    SKILL.md
```

## Лицензия

См. репозиторий matreshka-whisper / политику matreshka-dev.
