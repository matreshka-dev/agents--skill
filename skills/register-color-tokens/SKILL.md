---
name: register-color-tokens
description: >-
  Registers Matreshka BFF color tokens in ClientSettings after creating or using
  them in UI. Use when adding or editing color token definitions, wiring
  colors: { [ColorRole.*]: ... } in matreshka config, or when the client throws
  "Color token ... not found".
---

# Регистрация color token (Matreshka BFF)

## Главное правило

**Создать файл токена недостаточно.** Любой `ColorToken`, который передаётся в `colors` компонента BFF, должен быть **зарегистрирован** в `clientSettings.colors`. Иначе клиент не получит палитру при handshake и упадёт с `Color token <id> not found`.

## Роли файлов (в конфиге Matreshka приложения)

| Шаг | Что искать в проекте |
|-----|----------------------|
| Определение токена | Модуль(и) с экспортом `*ColorToken` (обычно каталог вроде `color-tokens/`) |
| Регистрация | Объект **`clientSettings`** / **`ClientSettings`** — массив **`colors`** |
| Использование в UI | `colors: { [ColorRole.Background]: myColorToken, ... }` на страницах и в общих компонентах конфига |

Общие переходы для ролей (shared `transition`, например `colorRoleTransition`) — это **не** отдельная палитра, в `colors` их не добавляют.

## Порядок при новом токене

1. Создать токен (экспорт `*ColorToken: ColorToken`, при необходимости `transition: colorRoleTransition`).
2. **Импортировать** токен в модуль, где объявлен `clientSettings`.
3. **Добавить** объект токена в `clientSettings.colors` (рядом с родственными: фоны с фонами, текст с текстом).
4. Использовать тот же **экземпляр** константы в компонентах (не дублировать объект палитры вручную).
5. После правок — перезагрузка клиента / новый handshake (dev-сервер backend подхватывает сам).

## Чеклист агента

При добавлении или изменении цветов в конфиге Matreshka:

- [ ] Каждый новый `*ColorToken` есть в `clientSettings.colors`
- [ ] Импорт в модуль `clientSettings` соответствует имени константы
- [ ] Нет «висячих» токенов: если токен объявлен, но не в `colors` — либо зарегистрировать, либо не использовать в UI

## Пример

```typescript
// определение токена
export const bgAccentPrimaryTintColorToken: ColorToken = {
  light: { ... },
  dark: { ... },
  transition: colorRoleTransition,
};

// clientSettings
import { bgAccentPrimaryTintColorToken } from '...';
// ...
colors: [
  // ...
  bgAccentPrimaryTintColorToken,
],

// компонент BFF
colors: { [ColorRole.Background]: bgAccentPrimaryTintColorToken },
```

## Как устроен id

Клиент идентифицирует палитру по хешу содержимого (`colorTokenId` / SHA-1 от JSON токена). Регистрация в `clientSettings.colors` как раз отдаёт эти определения клиенту; без записи в массиве id в UI не резолвится.
