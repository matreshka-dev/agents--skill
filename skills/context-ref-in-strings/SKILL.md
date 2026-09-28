---
name: context-ref-in-strings
description: >-
  Embeds Matreshka BFF ContextRef in template strings via toString() for
  placeholders, nested ref paths, and strictRef for cross-context segments.
  Use when mixing static text with context in text(), link.value, or fixing
  broken dynamic paths.
---

# `ContextRef` в строках (Matreshka BFF)

## Зачем

`ContextRef` — не значение, а **ссылка на путь** в контексте. Чтобы вставить её в **шаблонную строку** (`value` у `text()`, `link.value`, `button.link` и т.п.) как **плейсхолдер** `@{<id>.<path>}`, ref нужно превратить в строку через **`toString()`**.

Без явного `toString()` код легко читается как «подставить значение сейчас», а сериализатор ожидает **плейсхолдер**, который резолвится на клиенте при изменении контекста.

## Базовый паттерн

```typescript
text(`Ваше имя: ${this.context.ref("name").toString()}`);
```

```typescript
button(
  {
    link: {
      value: `/users/${this.context.ref("userId").toString()}`,
    },
  },
  [text("Открыть профиль")],
);
```

**Правило:** внутри `` `…${…}…` `` для каждого участка из контекста — **`context.ref('…').toString()`** (или цепочка ref, см. ниже, с **`toString()` на итоговом ref**).

Статический текст + одно поле контекста можно также выразить как `ref` без строки; **смешанный** текст (префикс, суффикс, несколько полей, URL) — через шаблон и `toString()`.

## Плейсхолдеры внутри путей (один контекст)

Путь можно **продолжать** через `ref()` — в том числе когда следующий сегмент задаётся **другим ref** того же контекста (динамический индекс или вложенный ключ):

```typescript
// userId хранит индекс; имя берётся из users[userId].name — всё один Context
text(
  `Пользователь: ${this.context.ref("users").ref(this.context.ref("userId")).ref("name").toString()}`,
);
```

Итоговый ref в строке — **один**; **`toString()` один раз** на конце цепочки.

## Кросс-контекстные сегменты: `strictRef`, не `ref`

Если сегмент пути должен браться из **другого экземпляра `Context`**, обычная склейка через `ref(другойRef)` **не даёт** корректной строгой типизации значения сегмента. Используй **`strictRef(otherRef)`** — в путь попадает плейсхолдер второго контекста (`other.toString()`), тип сегмента выводится из **`PathValue<OtherContext, OtherPath>`**.

```typescript
const dynamicTitleRef = cardsContext
  .ref("cards")
  .strictRef(indexContext.ref("key"));

text(`Карточка: ${dynamicTitleRef.toString()}`);
```

- **`ref()`** — пути и продолжения **внутри одного** контекста (и ref того же context на следующий сегмент).
- **`strictRef()`** — сегмент пути из **другого** контекста; для типов значений сегмента — **`strictRef`**, не `ref`.

Общая типизация ref — скилл [strict-context-ref](../strict-context-ref/SKILL.md).

## Чеклист агента

- [ ] В шаблонных строках с данными контекста у ref есть **`.toString()`**
- [ ] Динамический сегмент в **том же** контексте — цепочка **`.ref(…).ref(otherRef)…`**, затем **`.toString()`**
- [ ] Сегмент из **другого** `Context` — **`.strictRef(other.ref('…'))`**, не `ref(otherRef)` ради типов
- [ ] Не путать с **`ref` prop** компонента (передаётся `ContextRef` целиком, без шаблонной строки)

## Антипаттерны

```typescript
// Смешанный текст без плейсхолдера — ref в строке без toString() (неявно может сработать,
// но в проекте и в доках — всегда явный toString())
text(`Привет, ${this.context.ref("name")}`);

// Кросс-контекстный сегмент через ref — TS не свяжет тип сегмента с другим контекстом
cardsContext.ref("cards").ref(indexContext.ref("key"));

// Ожидание «значения сейчас» в BFF вместо плейсхолдера
text(`Имя: ${this.context.value("name")}`); // value() — для серверной логики, не для клиентского binding в строке
```
