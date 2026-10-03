---
name: rules-not-conditions-for-loading-ui
description: >-
  Uses Matreshka BFF component rules and overrides for loading button labels
  instead of hiding the control with conditions. Use when changing submit button
  text during async work—not for the full form submit pattern.
---

# Loading UI через `rules`, не через `conditions` (Matreshka BFF)

## Правило

| Механизм | Эффект |
| --- | --- |
| **`conditions`** у компонента | Показать узел или **убрать** из дерева |
| **`rules`** | Узел **остаётся**; при выполнении условий — **`overrides`** props |

Для кнопки submit/async:

- не делай **две кнопки** («Отправить» / «Загрузка») через взаимоисключающие `conditions`;
- меняй текст или вид через **`rules`** + `overrides.content` (и другие props при необходимости).

Мгновенный флаг loading, `form`, `ServerAction` и общий handler — **`prevent-duplicate-form-submit`**. Мгновенная запись флага на клиенте — **`instant-ui-set-context-value`**.

## Шаблон (только визуал)

```typescript
const loadingRef = this.context.ref("loading");

button(
  {
    rules: [
      {
        conditions: [when.equals(loadingRef, true)],
        overrides: {
          content: [text("Загрузка")],
        },
      },
    ],
  },
  [text("Отправить")],
);
```

`onClick` / `onSubmit` и установка `loading` — в **`prevent-duplicate-form-submit`**, не копируй полный submit-сценарий здесь.

## Антипаттерн

```typescript
// ❌ Две кнопки — ломает единый submit/form flow
button(
  { conditions: [when.equals(loadingRef, false)], onClick: submit },
  [text("Отправить")],
);
button(
  { conditions: [when.equals(loadingRef, true)] },
  [text("Загрузка")],
);
```

## Чеклист

- [ ] Одна кнопка видима во время loading
- [ ] Текст/вид — через `rules`
- [ ] Полный async submit — **`prevent-duplicate-form-submit`**
