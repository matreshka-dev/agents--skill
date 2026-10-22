---
name: context-init-in-boot
description: >-
  Chooses Matreshka BFF Context preload, lazy init, or await context.init() in
  entry boot() when content/overlays use context.get(); refs-only UI often
  skips manual init. Use for Page/Dialog Context, «not initialized» errors, or
  designing context loading.
---

# Инициализация `Context` на entry (Matreshka BFF)

## Три режима

| Режим | Настройка | Когда |
| --- | --- | --- |
| **Lazy** | только `data` | По умолчанию; первый запрос клиента к контексту |
| **Preload** | `preload: true` | Данные нужны сразу при авторизации клиента |
| **Ручной `init()`** | `await this.context.init()` в `boot()` | Сервер строит `content()` / `overlays()` через **`get()`** или ветки от загруженных данных |

### Практическое правило

- UI только на **`ref()`** + **`when.*`** → часто **без** ручного `init()` в `boot()` (lazy или preload по смыслу).
- **`context.get()`**, spread массива, `if` от данных в **`content()`** / **`overlays()`** → **`await context.init()` в `boot()`** (обязательно).

Корзина/badge на всех маршрутах — **`client-scoped-context-for-shared-ui`**, не дублируй `init()` в каждом виджете.

## Когда нужен `init()` в `boot()`

Если в **методах рендера** entry (`content()`, `overlays()`) вызывается:

- `this.context.get(...)` или ветвление (`if`, spread);
- построение дерева от уже загруженных полей (не только `ref` в props);

контекст должен быть **загружен до сериализации** через `await this.context.init()` в **`boot()`**.

## Почему так

Порядок при открытии страницы (`bootServerComponents`):

1. `await page.boot()`
2. `page.serialize()` → `initProperties()` → **рендер (`content()` и т.п.)**

Без `init()` в `boot()` вызовы `get` / `set` / `watch` до создания хранилища → `Context … is not initialized`. Подробности — JSDoc у `Context.init()` в `@matreshka/bff/core`.

## Шаблон `boot()`

```typescript
override async boot(): Promise<unknown> {
  await super.boot();
  await this.context.init();
  return;
}
```

После `init()` можно синхронно нормализовать данные в `boot()`, до `serialize()`.

Привязка `Context` к entry на выходе — **`context-destroy-with-entry`**.

## Примеры

```typescript
// Lazy + refs — init в boot не обязателен
protected content() {
  return [
    textInput(this.context.ref("name")),
    text(
      {
        conditions: [when.notEquals(this.context.ref("greeting"), "")],
      },
      `Значение: ${this.context.ref("greeting").toString()}`,
    ),
  ];
}
```

```typescript
// get в рендере — init обязателен
protected content() {
  const items = this.context.get("items");
  return items.length ? [forEach({ /* … */ })] : [text("Пусто")];
}
```

```typescript
context = new Context({
  preload: true,
  data: async () => ({ cart: [] }),
});
```

## Антипаттерн

```typescript
protected content() {
  const items = this.context.get("items"); // ❌ без await context.init() в boot()
  return items.length ? [...] : [...];
}
```

Не дублируй загрузку `data` внутри `content()`.

## Dialog

Тот же принцип для entry-диалогов с `Context`: **`await context.init()` в `boot()`**, если рендер читает `value`.

## Чеклист агента

- [ ] `get()` в рендер-методах entry → `init()` в `boot()`
- [ ] Только refs/conditions → lazy или preload; без лишнего `init()`
- [ ] `await super.boot()` первым при переопределении `boot`
- [ ] Асинхронная загрузка в `Context.data`, не в `content()`

## Связанные скиллы

- **`prefer-context-ref-in-ui`** — когда обойтись без `get()` в props UI
- **`context-destroy-with-entry`** — destroy при уходе entry
