---
name: context-date-iso-timezone
description: >-
  Serializes Date values for Matreshka BFF Context as ISO 8601 strings with an
  explicit timezone (Date#toJSON or toISOString) for datetime(), projectDateField, and
  native pickers. Use when wiring dates into Context, forms, boot/data loaders,
  or copying Date fields from Prisma/API into context values.
---

# Даты в `Context` (Matreshka BFF)

## Правило

Поля контекста для даты/времени — **`string`**, не `Date`. На сервере момент из `Date` (Prisma, `new Date(...)`) **сериализуй в ISO 8601 с явным поясом** (`Z` или смещение `+03:00`), чтобы клиент однозначно парсил через `new Date(value)`.

Используй **`date.toJSON()`** или **`date.toISOString()`** — для `Date` это одно и то же (ISO UTC с `Z`):

```typescript
const iso = dbDate.toISOString();
context.setValue('bornAt', iso);
// или при сборке data() / map*ToFormValues:
bornAt: user.birthday ? user.birthday.toJSON() : '',
```

## Почему не «только день»

- **`toISOString().slice(0, 10)`** и голый **`YYYY-MM-DD`** без пояса — плохой дефолт для контекста: теряется привязка к моменту, календарный день на клиенте может сдвинуться.
- Строка **без** пояса вроде `"2026-06-25T11:30:00"` клиент может трактовать как **локальное время устройства** — момент неоднозначен.

Хорошо: `"2026-06-25T08:30:00.000Z"`, `"2026-06-25T11:30:00+03:00"`.

## Где это важно

- `datetime(..., context.ref('…'))`, `projectDateField({ valueRef })`, `showDatePicker(..., ref)`.
- `preload` / `data()` диалогов и страниц: значения уходят клиенту как JSON-строки.
- Строка из metadata/БД перед показом в форме: если это не ISO с поясом — распарси (`Date.parse`) и снова **`toISOString()`** / **`toJSON()`**, не `slice(0, 10)`.
- В Prisma из контекста — `new Date(trimmed)` после валидации; правила домена (только день vs момент) — отдельно от формата строки в UI.

## Чеклист

- [ ] Новые поля даты в `Context` — тип **`string`**
- [ ] Из `Date` в контекст — **`toJSON()`** / **`toISOString()`**, не `slice(0, 10)`
- [ ] Не кладёшь в контекст «голый» datetime без `Z` / offset, если нужен точный момент
- [ ] UI — **`context.ref('field')`**, не `value()` в дереве ([prefer-context-ref-in-ui](../prefer-context-ref-in-ui/SKILL.md))

## Антипаттерны

```typescript
// Плохо: UTC-день без времени и пояса в строке для пикера
bornAt: user.birthday.toISOString().slice(0, 10);

// Плохо: Date в JSON контекста (не сериализуется предсказуемо для клиента)
data: { bornAt: user.birthday };
```

## Справка Matreshka

Раздел **`datetime()`** в reference components: ISO 8601, важность `Z` / offset, парсинг `new Date(value)` на клиенте (`matreshka/docs/reference/components-list.md`).
