---
name: storage-vs-context
description: >-
  Chooses Matreshka client storage versus BFF Context for auth tokens,
  tenant ids, and UI state. Use when persisting data, route guards, or
  designing page state.
---

# `storage` vs `Context` (Matreshka BFF)

## Правило

| | **`client.storage`** | **`Context`** |
| --- | --- | --- |
| Где живёт | Клиент (snapshot в `client-state`) | BFF, sync с клиентом |
| Жизненный цикл | Между сессиями / reload | Сценарий страницы, формы, списка |
| Типичное использование | JWT, tenant id, onboarding flags | Поля формы, списки, loading UI |
| Доступ в router | `client.storage.get("jwt")` | Обычно после создания `Page` |

- **Context** — состояние текущего UI-сценария.
- **storage** — данные клиента, нужные **до** построения страницы или после перезагрузки.

## Шаблоны

```typescript
// Guard до Page
app.router.addPage("/admin", async (client) => {
  const jwt = client.storage.get("jwt");
  const session = await authService.findSessionByJwt(jwt);
  return session?.role === "admin" ? new AdminPage() : undefined;
});

// Запись токена с BFF
client.storage.set("jwt", token);
```

```typescript
// Состояние экрана
class ProfilePage extends Page {
  context = new Context({
    data: async () => ({ name: "", editing: false }),
  });
}
```

## Антипаттерны

```typescript
// ❌ JWT только в Context, если router должен решать доступ до Page
// ❌ Черновик формы в storage, если не нужна персистентность между визитами
```

## Чеклист

- [ ] Персистентность на клиенте / pre-route → `storage`
- [ ] Редактируемый UI и sync → `Context` + `ref`
- [ ] Guard → **`route-guard-return-undefined`**
