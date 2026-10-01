---
name: context-destroy-with-entry
description: >-
  Binds Matreshka BFF Context to Page or Dialog via stopUsing$: on leave or
  close, call context.destroy() if not already destroyed. Without this,
  subscriptions and background updates outlive the entry.
---

# Уничтожение контекста при завершении entry (Page / Dialog)

## Правило

Привязать `Context` к **entry-component** (`Page`, `Dialog` и другим корневым компонентам маршрута/модалки) через **`stopUsing$`**: при уходе со страницы или закрытии диалога вызывать **`context.destroy()`**, если контекст ещё не уничтожен (`!context.isDestroyed()`).

Без этой привязки подписки контекста, незавершённая загрузка (`init`) и фоновые обновления могут жить дольше, чем entry на клиенте.

## Когда применять

- Добавляешь или правишь **Page** / **Dialog** с собственным `Context` (не только глобальный client context).
- В `boot()` или server actions вешаешь **долгоживущие подписки** на контекст или данные, привязанные к странице.
- На одной entry несколько контекстов (списки, вложенные блоки) — **каждый** нужно привязать к тому же entry (или к своему дочернему entry, если контекст локален ему).

С **`context-init-in-boot`**: сначала `await context.init()` в `boot()` там, где рендер читает `value()`; destroy по `stopUsing$` — парная операция на выходе.

## Почему `stopUsing$`

У компонента BFF (`Component`, в том числе `Page` / `Dialog`):

- **`startUsing$`** — появилось первое client-instance entry;
- **`stopUsing$`** — снято последнее instance (уход со страницы, закрытие диалога, размонтирование).

Это сигнал «entry больше не используется на клиенте» — подходящий момент для `Context.destroy()`.

`Context.destroy()` снимает подписки, завершает `data$`, шлёт клиентам `ContextDestroyMessage` и снимает контекст с реестра (см. `@matreshka/bff/core`).

## Шаблон (утилита в проекте)

```typescript
import type { Context } from '@matreshka/bff/core';
import type { JsonObject } from '@matreshka/shared/types/json';
import type { Observable } from 'rxjs';

type DestroyableEntry = {
  stopUsing$: Observable<void>;
};

export function destroyContextWithEntry<T extends JsonObject>(
  entry: DestroyableEntry,
  context: Context<T>,
): Context<T> {
  entry.stopUsing$.subscribe(() => {
    if (!context.isDestroyed()) {
      context.destroy();
    }
  });

  return context;
}
```

## Шаблон (Page / Dialog)

```typescript
export class ExamplePage extends Page {
  context = destroyContextWithEntry(
    this,
    new Context({ /* preload, data, initDataCallback */ }),
  );

  async boot(): Promise<unknown> {
    await super.boot();
    await this.context.init(); // если нужен context-init-in-boot
    return;
  }
}
```

Удобная обёртка на базовом классе страниц:

```typescript
protected entryContext<T extends JsonObject>(context: Context<T>): Context<T> {
  return destroyContextWithEntry(this, context);
}

// ...
context = this.entryContext(new Context({ ... }));
```

## Dialog

Тот же вызов: **`destroyContextWithEntry(this, context)`** в entry-диалоге, где `this` — экземпляр `Dialog`.

## Чеклист агента

- [ ] У entry есть `Context`, живущий только пока открыта страница/диалог
- [ ] На `entry.stopUsing$` вызывается `context.destroy()` с проверкой `isDestroyed()`
- [ ] Несколько контекстов на одной entry — у каждого своя привязка (или общий helper)
- [ ] Server actions после destroy не обращаются к контексту без проверки (см. `return-server-action-promise` и `isDestroyed()` в `finally`)

## Антипаттерн

```typescript
export class LeakyPage extends Page {
  context = new Context({ preload: true, /* ... */ });
  // ❌ нет подписки на this.stopUsing$ → destroy не вызывается при уходе
}
```

## Замечания

- Авто-`destroy` при отключении **всех** клиентов от контекста — отдельный механизм реестра; привязка к **entry** нужна, когда lifecycle страницы/диалога должен явно завершать контекст, не дожидаясь отвязки клиентов.
- Подписка на `stopUsing$` без `takeUntil` обычно допустима: entry и контекст уничтожаются вместе; если в проекте есть общий стиль отмены подписок — следуй ему, сохраняя вызов `destroy()` в обработчике.
