---
name: client-scoped-context-for-shared-ui
description: >-
  Stores session-wide UI data (badges, counters, cart) in one Context per Client
  instead of new Context per reusable shell widget. Use for header/shell on many
  pages, or when the same remote data reloads on every navigation. Pair with
  context-plan-destroy-on-create for lifecycle.
---

# Client-scoped контекст для общего shell UI

## Правило

Данные, которые **одни и те же для пользователя на всех страницах** (счётчик уведомлений, корзина, профиль в шапке, глобальные флаги сессии), храни в **одном контексте на `Client`**, а не в `new Context()` внутри переиспользуемого компонента (кнопка, badge, виджет в layout).

Переиспользуемый UI только **читает** поля через `ContextRef` (см. **`prefer-context-ref-in-ui`**) или получает ref снаружи; **первичная загрузка** — у контекста сессии, один раз на клиента.

## Когда применять

- Виджет в **шапке / sidebar / shell**, который вставляется на **много страниц** (колокольчик, корзина, аватар).
- При **каждой навигации** снова вызывается `context.init()` / тот же API для одних и тех же данных.
- Несколько экранов должны **мгновенно видеть одно значение** (прочитал уведомление на одной странице — badge на другой обновился без повторного полного fetch layout).

## Когда оставить контекст у entry (Page / Dialog)

- Данные **живут только на маршруте**: список заказов, форма редактирования, деталка сущности.
- Контекст привязан к **`stopUsing$` страницы/диалога** — см. **`context-destroy-with-entry`**.

Сессионный контекст и page-контекст **дополняют** друг друга: в page-контексте — локальное состояние экрана, в client-scoped — то, что нужно shell по всему приложению.

**Не** используй **`persistent: true`** для session shell: нужен **один Context на `Client`** (`WeakMap`), память масштабируется с клиентами. `persistent` — **один** контекст на процесс для process-wide данных; см. **`context-plan-destroy-on-create`**.

### Узкая зона приложения (не вся сессия)

Тот же **один Context на `Client`**, но destroy при **уходе с маршрута / раздела** — `destroy…Context()` из route guard или базовой страницы зоны. Подписки — `takeUntil(client.destroy$)`. См. **`context-plan-destroy-on-create`**, **`route-guard-return-undefined`**.

## Антипаттерн

Переиспользуемый компонент создаёт свой контекст и грузит данные при каждом экземпляре:

```typescript
export class NotificationBell extends Button {
  constructor(options: NotificationBellOptions = {}) {
    const context = new Context<{ unreadCount: number }>({
      preload: true,
      data: async () => ({ unreadCount: await countUnread() }),
    });
    void context.init();

    super({
      /* badge и иконки через context.ref('unreadCount') */
      onClick: options.onClick,
    });

    this.stopUsing$.subscribe(() => context.destroy());
  }
}
```

Проблемы:

1. **Новый `Context` на каждую страницу** — при монтировании shell снова `init()` и повторный запрос к API.
2. **`destroy` по `stopUsing$` кнопки** — контекст живёт ровно пока виден один экземпляр виджета; при уходе со страницы данные выбрасываются, на следующей странице — снова загрузка.

Типичный симптом: один и тот же badge/counter «мигает» или дергает сеть при каждом переходе между разделами.

## Рекомендуемый паттерн

### 1. Один контекст на клиента (getter + кэш)

```typescript
import { Client, Context, currentClient } from '@matreshka/bff/core';

export type SessionUserContextValues = {
  unreadNotifications: number;
  // cart, profile snippet, …
};

const CACHE = new WeakMap<Client, Context<SessionUserContextValues>>();

export function currentSessionUserContext(): Context<SessionUserContextValues> {
  const client = currentClient();
  let context = CACHE.get(client);
  if (context === undefined) {
    context = new Context<SessionUserContextValues>({
      preload: true,
      data: async () => ({
        unreadNotifications: await loadUnreadCount(),
      }),
    });
    CACHE.set(client, context);
  }
  return context;
}

export function destroySessionUserContext(client: Client = currentClient()): void {
  const context = CACHE.get(client);
  if (context === undefined) {
    return;
  }
  CACHE.delete(client);
  if (!context.isDestroyed()) {
    context.destroy();
  }
}
```

`WeakMap<Client, Context>` — контекст не переживает отключение клиента; явный `destroySessionUserContext` вызывают при logout / disconnect, если в проекте есть единая точка завершения сессии.

### 2. Инициализация один раз в shell entry

В базовой **layout-странице** или корне приложения для залогиненного пользователя:

```typescript
async boot(): Promise<unknown> {
  await super.boot();
  const session = currentSessionUserContext();
  await session.init(); // см. context-init-in-boot, если рендер читает value()
  return;
}
```

### 3. Переиспользуемый компонент без своего Context

```typescript
export class NotificationBell extends Button {
  constructor(options: NotificationBellOptions = {}) {
    const unreadRef = currentSessionUserContext().ref('unreadNotifications');

    super({
      onClick: options.onClick,
      overlays: [/* badge через unreadRef, when.notEquals(unreadRef, 0) */],
      content: [/* иконки с when.equals / when.notEquals */],
    });
    // ❌ не создавать Context, init и destroy здесь
  }
}
```

Опционально: передавать `unreadRef` параметром, если виджет используют и в зонах без session context (гостевой режим) — тогда снаружи решают, откуда ref.

### 4. Обновление после действий пользователя

На странице списка уведомлений после «прочитать всё»:

```typescript
currentSessionUserContext().setValue('unreadNotifications', 0);
```

Так badge в шапке обновится **без** повторного mount shell и без второго контекста.

## Связанные скиллы

| Скилл | Роль |
| ----- | ---- |
| `context-init-in-boot` | `await session.init()` в shell до `value()` в рендере |
| `context-destroy-with-entry` | destroy **page/dialog** контекстов, не session |
| `prefer-context-ref-in-ui` | refs из session context в props виджетов |
| `strict-context-ref` | типы ref для полей session context |

## Чеклист агента

- [ ] Данные нужны на **многих маршрутах** одного залогиненного клиента
- [ ] Нет `new Context` в конструкторе **переиспользуемого** shell-виджета
- [ ] Есть getter `current…Context()` с **одним экземпляром на `Client`**
- [ ] `init()` — в **shell / layout boot**, не в конструкторе виджета
- [ ] Виджет использует **refs** session context; действия на детальных экранах **пишут** в тот же context
- [ ] Session context уничтожается при **logout/disconnect**, а не при каждом `stopUsing$` виджета

## Замечания

- Не складывай в session context **объёмные списки**, нужные только одной странице — это раздувает память и сериализацию на всех маршрутах.
- Если поле session context зависит от **выбранного пространства / tenant**, обновляй его при смене selection в том же client-scoped контексте (или в nested поле), а не заводи второй контекст на виджет.
- Имя `currentSessionUserContext` / `currentAppUserContext` — **конвенция проекта**; суть скилла — scope по `Client`, а не конкретное имя функции.
