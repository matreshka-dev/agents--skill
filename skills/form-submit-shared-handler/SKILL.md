---
name: form-submit-shared-handler
description: >-
  Wraps Matreshka BFF form fields and confirmation controls in form() and
  reuses one handler for form onSubmit and the confirmation button onClick.
  Use when creating, editing, or reviewing forms, inputs with a confirmation
  action, or submit flows for web and mobile.
---

# Общий обработчик подтверждения формы (Matreshka BFF)

## Правило

При создании формы:

1. Оборачивай поля и кнопку подтверждения в компонент `form`.
2. Выноси подтверждение в одну функцию.
3. Передавай эту же функцию в `onSubmit` формы и `onClick` кнопки подтверждения.

Это два сценария подтверждения одной формы:

- `onSubmit` обрабатывает Enter, action мобильной клавиатуры и стандартный submit;
- `onClick` обрабатывает явное нажатие кнопки.

Оба сценария должны запускать одну и ту же логику.

## Шаблон

```typescript
const submitForm = () => {
  const name = this.context.value("name");
  this.context.setValue("result", `Привет, ${name}`);
};

form(
  {
    onSubmit: submitForm,
  },
  [
    textInput(this.context.ref("name")),
    button({ onClick: submitForm }, [text("Отправить")]),
  ],
);
```

## Не делать

```typescript
// ❌ Поля и подтверждение не объединены компонентом form
column([
  textInput(this.context.ref("name")),
  button({ onClick: submitForm }, [text("Отправить")]),
]);

// ❌ Submit и клик выполняют разную логику
form(
  { onSubmit: submitForm },
  [
    textInput(this.context.ref("name")),
    button({ onClick: confirmByClick }, [text("Отправить")]),
  ],
);
```

Не копируй тело обработчика в два callback: используй одну ссылку на функцию, чтобы сценарии не расходились при последующих изменениях.

## Чеклист агента

- [ ] Поля и кнопка подтверждения находятся внутри `form`
- [ ] Основная логика подтверждения вынесена в общий обработчик
- [ ] `form.onSubmit` ссылается на общий обработчик
- [ ] `button.onClick` ссылается на тот же обработчик
- [ ] Для submit и клика нет двух независимых реализаций одной операции
