---
name: prefer-local-actions
description: >-
  Prefers Matreshka client-only LocalAction over server handlers when an
  interaction only changes UI or invokes a client API. Covers component.local,
  instance.local, Context/ContextRef local actions, and global local actions.
  Use when adding or reviewing BFF event actions, component commands, context
  updates, animation, clipboard, or other client-side UI behavior.
---

# Локальные действия Matreshka BFF

## Основное правило

Если действие не требует BFF, backend, бизнес-правил, авторизации или серверного
результата, предпочитай `LocalAction` серверному обработчику. Локальное действие
исполняется client runtime сразу, без `server-interaction` и без ожидания
round-trip.

Используй server handler, когда нужно:

- выполнить backend или асинхронную серверную операцию;
- проверить права, бизнес-инварианты или актуальные данные на BFF;
- сохранить данные на сервере;
- получить серверный результат или обработать серверную ошибку.

`conditions` у `LocalAction` — только клиентская фильтрация перед выполнением.
Они не являются проверкой безопасности или бизнес-правилом.

## Где искать локальное API

### Контекст и ContextRef

Для мгновенного изменения клиентского зеркала контекста используй:

```ts
button({ onClick: this.context.local.set({ loading: true }) }, [
  text("Загрузить"),
]);

button({ onClick: this.context.local.toggle("enabled") }, [
  text("Переключить"),
]);

button({ onClick: this.context.local.inc("quantity") }, [text("+1")]);
button({ onClick: this.context.local.dec("quantity") }, [text("-1")]);
```

Если есть `ContextRef`, локальное действие можно вызвать на нём:

```ts
const enabledRef = this.context.ref("enabled");
const quantityRef = this.context.ref("quantity");

button({ onClick: enabledRef.local.toggle() }, [text("Переключить")]);
button({ onClick: quantityRef.local.dec() }, [text("Уменьшить")]);
```

- `context.local.set(patch)` поддерживает dotted paths и несколько полей.
- `ref.local.set(value)` записывает значение конкретного ref.
- `toggle` предназначен для boolean-полей.
- `inc`/`dec` предназначены для числовых полей; шаг по умолчанию равен `1`.
- Обновление происходит сразу на клиенте, после чего клиент синхронизирует
  изменение контекста с BFF.

Не заменяй `context.set` или `ref.set` на `.local`, если изменение должно
произойти именно на BFF в серверном обработчике.

### Компонент: два уровня адресации

У component local API есть два уровня:

1. `component.local.*` — действие адресовано базовому id компонента и
   выполняется у всех его активных instances **в текущем клиенте**. Это
   client-only broadcast внутри одного client runtime, откуда пришёл
   interaction; другие подключённые клиенты его не получают.
2. `instance.local.*` — действие адресовано `instance.id` и выполняется только
   у конкретного DOM-инстанса.

В обработчике `instance` приходит из события:

```ts
const map = new Map({
  apiKey: process.env.MAPS_KEY!,
  center: { latitude: 55.75, longitude: 37.62 },
  zoom: 12,
});

button({
  onClick: ({ instance }) => [
    instance.local.setZoom(8),       // только карта, по которой кликнули
    instance.component.local.zoomIn(), // все instances этого map-компонента
  ],
}, [text("Масштаб")]);
```

До клика, когда нужен известный инстанс, используй `createInstance()` и положи
каждый созданный `ComponentInstance` в дерево только один раз.

Типовые локальные методы компонентов:

- `map.local.setCenter`, `setZoom`, `zoomIn`, `zoomOut`;
- `board.local.setCenter`, `setZoom`, `zoomIn`, `zoomOut`;
- `stack.local.scrollTo`;
- `camera.local.switchCamera`;
- команды `textEditor.local`;
- `dialog.local.close` и `popover.local.close`.

Параметры component local API обычно совпадают с соответствующими серверными
командами. Локальному методу дополнительно можно передать `ActionConfig` с
`conditions`.

Для закрытия entry выбирай область явно:

```ts
instance.local.close();           // один dialog/popover
instance.component.local.close(); // все instances этого компонента в текущем клиенте
```

Это отличается от серверной команды `component.close()`: она выполняется на BFF
и может отправить команду инстансам компонента на разных подключённых клиентах
(или только явно переданным instances).

Не путай адресацию:

- `Component` — BFF-объект с базовым `id`;
- `ComponentInstance` — одно появление компонента на клиенте с `instance.id`.

### Глобальные локальные действия

Глобальный объект `local` используют для действий, не принадлежащих одному
компоненту:

```ts
import { local } from "@matreshka/bff/core";

button({
  onClick: [
    local.writeTextToClipboard("Скопировано"),
  ],
}, [text("Скопировать")]);
```

`local.animate(...)` также является глобальным локальным действием; ему
передаётся полная конфигурация анимации (`duration` и `effects`). Глобальные
действия не имеют component target. Для анимации конкретного компонента можно
указать `componentId` или использовать его `component.local`/`instance.local`,
если такой метод доступен.

## Локальное и серверное действие вместе

Одно событие может выполнять несколько действий. Локальный UI-эффект можно
запустить сразу, а серверную работу оставить `ServerAction`:

```ts
button({
  onClick: [
    this.context.local.set({ loading: true }),
    async () => {
      await this.orders.submit();
    },
  ],
}, [text("Отправить")]);
```

Для полноценной формы с loading и защитой от повторной отправки используй
паттерн из `prevent-duplicate-form-submit`: локально меняй UI, но критические
проверки выполняй в серверном обработчике.

## Проверка перед реализацией

- [ ] Действие действительно client-only и не требует серверных данных?
- [ ] Для Context выбран `context.local` или существующий `ref.local`?
- [ ] Для компонента выбран правильный уровень: все instances или один?
- [ ] Если нужен один instance до события, создан отдельный `createInstance()`?
- [ ] `conditions` не используются как auth/business gate?
- [ ] Серверная операция, права и инварианты остаются в server handler?
- [ ] Для нескольких действий сохранён нужный порядок и типы обработчиков?

## Связанные скиллы

- `instant-ui-set-context-value` — подробные операции `context.local` и
  `ref.local`.
- `server-action-validate-in-handler` — проверки прав и инвариантов в BFF.
- `prevent-duplicate-form-submit` — loading и защита от повторной отправки формы.
