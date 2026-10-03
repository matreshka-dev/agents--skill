---
name: context-zod-draft-not-live-strict
description: >-
  Avoids overly strict Zod schemas on Matreshka BFF Context fields bound to
  live text inputs; uses draft fields or submit-time validation. Use when
  adding Context schema, forms, or filters with client context-values updates.
---

# Zod и «живой» ввод в `Context` (Matreshka BFF)

## Правило

`Context` с **`schema`** отклоняет невалидные **`context-values`** с клиента. Для полей, которые меняются **на каждый символ** (`textInput`, фильтры):

- не вешать сразу `z.string().email()` / жёсткие regex на bound ref;
- промежуточный ввод (`test@`) не пройдёт → поле «откатывается» с точки зрения BFF.

## Подходы

**1. Мягкое поле + валидация на submit**

```typescript
context = new Context({
  schema: z.strictObject({
    emailDraft: z.string(),
    email: z.string().email().optional(),
  }),
  data: async () => ({ emailDraft: "", email: undefined }),
});

// onSubmit: прочитать draft, setValue("email", validated) или показать ошибку
```

**2. Схема, допускающая промежуточные состояния**

```typescript
email: z.string(), // валидация формата только в server handler submit
```

**3. Разделение draft / validated**

Черновик в `*Draft`, итог после проверки — в отдельном ключе для бизнес-логики.

## Когда строгая schema уместна

- Поля, которые клиент **не** редактирует напрямую.
- Значения, приходящие только с BFF после проверки.
- Submit одним пакетом с уже собранными полями.

## Чеклист

- [ ] `textInput(ref)` + strict zod на том же ключе — пересмотреть
- [ ] Ошибки «не принимает ввод» → ослабить schema или draft
- [ ] Submit проверяет бизнес-правила явно
