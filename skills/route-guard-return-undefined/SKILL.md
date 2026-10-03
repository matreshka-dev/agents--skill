---
name: route-guard-return-undefined
description: >-
  Implements Matreshka BFF route guards by returning undefined to fall through
  to the next handler, plus wildcard fallback routes. Use when adding protected
  routes, auth checks, or 403/404 pages.
---

# Route guard: `undefined` и fallback (Matreshka BFF)

## Правило

1. Callback `addPage` может вернуть `undefined` — `Router` ищет **следующий** подходящий handler.
2. Guard на BFF: проверка доступа → `Page` или `undefined`.
3. Отдельный маршрут `**` (или цепочка) — для 403/404/forbidden.

Данные для guard **до** создания страницы (JWT, tenant) — **`client.storage`**, см. **`storage-vs-context`**.

## Шаблон

```typescript
app.router.addPage("/admin", async (client) => {
  if (!(await canAccessAdmin(client))) {
    return undefined;
  }
  return new AdminPage();
});

app.router.addPage("**", async () => new ForbiddenPage());
```

Параметры path:

```typescript
app.router.addPage("/users/{id}", async (_client, params) => {
  return new UserPage(params.id);
});
```

Wildcard группы:

```typescript
app.router.addPage("/docs/*", async () => new DocsSectionPage());
```

## Антипаттерны

```typescript
// ❌ throw вместо fall-through, если нужен следующий handler
app.router.addPage("/admin", async () => {
  if (!isAdmin) throw new Error("forbidden");
});

// ❌ JWT только из Context до создания Page — нужен storage
```

Возврат `new ForbiddenPage()` из guard **может** быть осознанным выбором, но тогда отдельный `**` не подхватит тот же сценарий — проектируй цепочку явно.

## Чеклист

- [ ] Нет доступа → `undefined` или согласованная Page-заглушка
- [ ] Есть fallback (`**` или следующий handler)
- [ ] Pre-route auth data → **`storage-vs-context`**
