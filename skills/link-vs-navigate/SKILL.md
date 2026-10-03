---
name: link-vs-navigate
description: >-
  Chooses Matreshka BFF link.value for real browser links versus
  currentClientPlatform().navigate() in onClick for server-decided navigation.
  Use when adding buttons, links, or routing from BFF handlers.
---

# `link` vs `navigate()` (Matreshka BFF)

## Правило

| Задача | Подход | Поведение в браузере |
| --- | --- | --- |
| Известный или собранный URL (в т.ч. с `ContextRef` в строке) | `link: { value: "…" }` | Настоящая ссылка: новая вкладка, «открыть в…» |
| Путь решается на BFF после логики (auth, флаги, сервисы) | `onClick` → `currentClientPlatform().navigate(path)` | Переход без `<a href>` |

## Шаблоны

```typescript
// Статический или предсказуемый URL
button({ link: { value: "/settings" } }, [text("Настройки")]);

// URL с плейсхолдером из контекста — всё ещё link
button(
  {
    link: {
      value: `/users/${this.context.ref("userId").toString()}`,
    },
  },
  [text("Профиль")],
);
```

```typescript
// Решение на сервере
button(
  {
    onClick: () => {
      const ok = this.context.value("isAuthorized");
      currentClientPlatform().navigate(ok ? "/dashboard" : "/login");
    },
  },
  [text("Продолжить")],
);
```

## Антипаттерн

```typescript
// ❌ Нужна была ссылка для UX браузера, но выбрали только navigate
button(
  { onClick: () => currentClientPlatform().navigate(`/users/${id}`) },
  [text("Профиль")],
);
```

## Чеклист

- [ ] Нужны middle-click / новая вкладка → `link.value`
- [ ] Путь зависит от серверной логики в момент клика → `navigate()`
- [ ] Динамический path в `link` — через `ref.toString()` в шаблонной строке (скилл **`context-ref-in-strings`**)
