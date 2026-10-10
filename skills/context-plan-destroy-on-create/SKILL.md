---
name: context-plan-destroy-on-create
description: >-
  When adding Matreshka BFF Context, plan context.destroy() from business
  lifecycle. Default contexts auto-destroy when last Client unbinds; manual
  destroy is an early memory optimization. persistent:true skips auto-destroy —
  then manual destroy is critical unless data lives for entire runtime. Use
  when creating Context or reviewing leaks.
---

# План уничтожения при создании Context

## Правило

При **`new Context(...)`** сразу зафиксируй **момент удаления** из runtime: кто владеет данными, когда сценарий закончен, и **кто вызывает `context.destroy()`** (с проверкой `context.isDestroyed()`).

## Default: auto-destroy при last client

У контекста **`persistent: false`** (по умолчанию) BFF **автоматически** вызывает `destroy()`, когда отвязан **последний** клиент (`authorizeClient` → `client.destroy$` → unbind). **Без привязанных клиентов контекст на BFF не остаётся** — «вечного» хранения нет.

**Ручной** destroy (entry `stopUsing$`, logout, инвалидация) — **оптимизация по времени**: освободить память и подписки **раньше**, чем уйдёт последний клиент. Это **не** единственный способ снять default-контекст, но **обязателен** для page-scoped state, пока сессия жива (навигация сама не уничтожает контекст).

**Навигация внутри одной сессии** сама контексты не снимает — отдельно от auto-destroy при disconnect.

## `persistent: true`

Флаг **`persistent`** (readonly на экземпляре) **отключает** auto-destroy при нуле клиентов. Контекст **переживает** период без потребителей; один **`data$` на процесс** для всех одновременно привязанных `Client`.

- **`persistent` ≠ immortal** — явный `context.destroy()` всё равно нужен, если данные **не** должны жить весь runtime (reload справочника, версия, shutdown).
- **Не** путать с **client-scoped** (`WeakMap<Client, Context>`): там память **растёт** с числом клиентов; `persistent` — один id, O(1) контекстов на процесс.
- **Не** использовать для персональных данных без строгой изоляции.

```typescript
new Context({
  persistent: true,
  preload: true,
  data: async () => loadGlobalCatalog(),
});
```

## Выбор момента destroy (по смыслу данных)

| Данные живут… | Момент destroy | Скилл / паттерн |
| ------------- | -------------- | ---------------- |
| **Страница / диалог** | `entry.stopUsing$` (раньше last client) | **`context-destroy-with-entry`** |
| **Сессия** пользователя (shell) | logout / `destroy…Context(client)`; `takeUntil(client.destroy$)` | **`client-scoped-context-for-shared-ui`** |
| **Раздел приложения** | route guard / уход с зоны | **`client-scoped-context-for-shared-ui`** + **`route-guard-return-undefined`** |
| **Process-wide** sync-кэш | `persistent: true` + **явный** destroy при инвалидации | этот скилл + `matreshka/docs/advanced/context-destroy-scenarios.md` |
| **Свой BFF-комponent** (store) | `component.stopUsing$` | advanced doc, component-scoped |
| Диалог **не открыли** | сразу `context.destroy()` | **`context-destroy-with-entry`** |

## Чеклист при добавлении Context

- [ ] Записано: «контекст X уничтожается, когда Y» (включая auto last-client для default)
- [ ] Page/Dialog — **`context-destroy-with-entry`**
- [ ] Session shell — один Context на **`Client`**, **не** `persistent` без причины
- [ ] `persistent: true` — обоснован process-wide; есть **ручной** destroy, если не весь runtime
- [ ] Component internal Context — destroy на **`stopUsing$` компонента**
- [ ] Подписки — `takeUntil(destroy$)` entry или `client.destroy$`
- [ ] После `await` — `isDestroyed()` / `ContextRef.setValue` в `finally`

## Антипаттерн

```typescript
// ❌ persistent без плана ручного destroy
export const cache = new Context({
  persistent: true,
  data: async () => loadHuge(),
});
// нет invalidate → утечка на BFF навсегда
```

```typescript
// ❌ Page context без раннего destroy (висит до last client, но last client может быть долго)
export class OrdersPage extends Page {
  context = new Context({ preload: true, data: async () => loadOrders() });
}
```

## Связанные скиллы

| Скилл | Когда |
| ----- | ----- |
| `context-destroy-with-entry` | Entry-scoped destroy |
| `client-scoped-context-for-shared-ui` | Один Context на Client |
| `context-init-in-boot` | Парный `init()` |
| `context-subscribe-take-until-destroy` | `data$` на entry |
| `storage-vs-context` | Context vs storage |

Документация: `matreshka/docs/advanced/context-destroy-scenarios.md`, `matreshka/docs/architecture/context-lifecycle.md`.
