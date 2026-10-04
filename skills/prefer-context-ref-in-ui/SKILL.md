---
name: prefer-context-ref-in-ui
description: >-
  Prefers Matreshka BFF ContextRef over context.value() when building UI trees
  so the client can react to context changes. Use when wiring outputs, inputs,
  conditions, forEach, or reviewing pages that read value() into text/value
  props.
---

# Ref вместо `value()` при генерации UI (Matreshka BFF)

## Идея

При **сборке дерева компонентов** (`content()`, хелперы верстки, `forEach` generator) данные из контекста лучше передавать как **`ContextRef`**, а не как **значение**, прочитанное через **`context.value(...)`** / **`ref.value()`** в момент сериализации.

| Подход                                                           | Что попадает в дерево           | Поведение на клиенте                                                          |
| ---------------------------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------- |
| **`ref('field')`** (prop `ref`, `when.*`, шаблон с `toString()`) | Привязка к **пути** в контексте | UI **обновляется**, когда контекст меняется (ввод, `setValue`, ответ сервера) |
| **`value('field')` в аргументах отображения**                    | **Снимок** на момент генерации  | Текст/`value` **заморожены**; смена контекста **не** перерисует этот фрагмент |

`value()` остаётся для **серверной логики** (обработчики, загрузка, ветвление структуры дерева), не для «показать поле контекста пользователю».

## Предпочтительно: ref

```typescript
text(this.context.ref("name"));
textInput(this.context.ref("draft"));
datetime(this.context.ref("createdAt"));

text({
  conditions: [when.defined(this.context.ref("subtitle"))],
  ref: this.context.ref("subtitle"),
});

forEach({
  ref: this.context.ref("items"),
  generator: ({ ref: itemRef }) => text(itemRef.ref("title")),
});
```

Смешанный текст — ref + плейсхолдер: скилл [context-ref-in-strings](../context-ref-in-strings/SKILL.md).

## Избегать в props отображения

```typescript
// ❌ имя зафиксировано при serialize
text(this.context.value("name"));
text(`Привет, ${this.context.value("name")}`);

forEach({
  ref: this.context.ref("items"),
  generator: ({ ref: itemRef }) => text(itemRef.value().title), // ❌ то же для каждого item
});
```

После `textInput` или `setValue` пользователь на клиенте **не увидит** обновление, если текст собран из `value()`.

## Когда `value()` уместен

**Не** в props «показать/редактировать поле», а там, где нужно **решение на BFF** в момент вызова:

- **`onClick` / `onSubmit` / сервисы** — прочитать, посчитать, вызвать API, `setValue`
- **Структура дерева** — разный набор блоков от длины списка, роли, feature-flag (тогда часто нужен `await context.init()` в `boot()` — скилл [context-init-in-boot](../context-init-in-boot/SKILL.md))
- **`boot()`** — нормализация данных до первой сериализации

```typescript
onSubmit: () => {
  const name = this.context.value("name"); // ✅ логика submit
  this.context.setValue("greeting", `Привет, ${name}`);
};
```

Для **видимости** блоков без пересборки всего дерева на сервере предпочитай **`when.*` на ref**, а не `if (this.context.value('loading'))` с разными статическими ветками, если достаточно показать/скрыть на клиенте.

## Чеклист агента

- [ ] Output/input/currency/datetime/image и т.п. — **`ref`**, не `value()` в аргументах
- [ ] `forEach` generator — **`itemRef.ref('…')`**, не `itemRef.value().…` для отображения
- [ ] Условия видимости — **`when.*(this.context.ref('…'), …)`**, где возможно
- [ ] `value()` в рендере только для **структуры** или с **`init()` в `boot()`**; осознанный trade-off «статичная ветка»
- [ ] Строки с контекстом — **`ref().toString()`**, не `value()`

## Связанные скиллы

- [context-ref-in-strings](../context-ref-in-strings/SKILL.md) — ref в шаблонных строках
- [strict-context-ref](../strict-context-ref/SKILL.md) — `ContextRef<C, P>` на странице
- [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md) — `ContextValueRef` в параметрах хелперов
- [context-init-in-boot](../context-init-in-boot/SKILL.md) — когда рендер всё же читает `value()`
