---
name: context-init-in-boot
description: >-
  Initializes Matreshka BFF Context in entry component boot() before render
  methods read context.value() in content/overlays. Use when adding Page or
  Dialog with Context, editing content() that uses context data, or fixing
  getValue / context not loaded errors during serialize.
---

# Инициализация контекста в `boot()` (entry component)

## Когда нужно

Если в **методах рендера** entry-component (например `content()`, переопределённый метод с деревом страницы, `overlays()`) вызывается:

- `this.context.value(...)` или чтение данных для ветвления (`if`, spread `...`);
- построение дерева, зависящее от уже загруженных полей контекста (не только `ref` в props);

то контекст должен быть **загружен до сериализации** через `await this.context.init()` в **`boot()`**.

## Почему так

Порядок при открытии страницы (серверный bootstrap Matreshka, `bootServerComponents`):

1. `await page.boot()`
2. `page.serialize()` → при первом обращении к `properties` вызывается `initProperties()` → **метод рендера (`content()` и т.п.)**

Рендер **не** вызывается в конструкторе страницы, но вызывается **до** ответа клиенту и **после** `boot()`. Без `init()` в `boot()` у `context.value()` нет `data$` → ошибки вроде `Cannot read properties of undefined (reading 'getValue')`.

Подробности — JSDoc у `Context.init()` в `@matreshka/bff/core`.

## Шаблон (Page и другие entry)

```typescript
async boot(): Promise<unknown> {
  await super.boot();
  await this.context.init();
  return;
}
```

После `init()` при необходимости можно синхронно поправить данные в контексте (дефолты, нормализация) — всё ещё внутри `boot()`, до `serialize()`.

Привязку `Context` к entry на выходе (destroy по `stopUsing$`) — скилл **`context-destroy-with-entry`**.

## Чеклист агента

- [ ] У entry-component есть `Context` и в рендер-методе используется `context.value` или логика от загруженных данных
- [ ] В `boot()` есть `await this.context.init()` (и `await super.boot()` первым, если переопределяете `boot`)
- [ ] Асинхронная загрузка остаётся в `Context` (`data` / `initDataCallback`), а не дублируется в `content()`

## Когда можно без `value()` в рендере

Если UI строится только на **`this.context.ref(...)`** и **`when.*`** (условия на клиенте), часто достаточно корректного `Context` и `preload`, но при любых **`context.value` в рендере** — **`init()` в `boot()` обязателен**.

**Без `value` в рендере:** ветки через `when.equals(someRef, …)` и refs в props — данные подтягиваются на клиенте по условиям.

**С `value` в рендере:** списки/spread от `context.value('items')`, флаги для `if`/`...` от `value` — нужны `boot` + `init`.

## Антипаттерн

```typescript
protected content() {
  const items = this.context.value('items'); // ❌ без await context.init() в boot()
  return items.length ? [...] : [...];
}
```

## Dialog

Тот же принцип для entry-диалогов с `Context`: перед сериализацией содержимого — **`await context.init()` в `boot()`** (или эквивалентный lifecycle entry), если рендер читает `value`.
