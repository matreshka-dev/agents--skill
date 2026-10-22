---
name: prefer-context-ref-in-ui
description: >-
  Prefers Matreshka BFF ContextRef over context.get() when building UI trees
  so the client can react to context changes. Use when wiring outputs, inputs,
  conditions, forEach, or reviewing pages that read get() into text/value
  props.
---

# Ref вместо `get()` при генерации UI (Matreshka BFF)

## Идея

При **сборке дерева компонентов** (`content()`, хелперы верстки, `forEach` generator) данные из контекста лучше передавать как **`ContextRef`**, а не как **снимок**, прочитанный через **`context.get(...)`** / **`ref.get()`** в момент сериализации.

| Подход                                                           | Что попадает в дерево           | Поведение на клиенте                                                          |
| ---------------------------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------- |
| **`ref('field')`** (prop `ref`, `when.*`, шаблон с `toString()`) | Привязка к **пути** в контексте | UI **обновляется**, когда контекст меняется (ввод, `set`, ответ сервера) |
| **`get('field')` в аргументах отображения**                      | **Снимок** на момент генерации  | Текст/`value` **заморожены**; смена контекста **не** перерисует этот фрагмент |

`get()` и `ref.get()` остаются для **серверной логики** (обработчики, загрузка, ветвление структуры дерева), не для «показать поле контекста пользователю».

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
text(this.context.get("name"));
text(`Привет, ${this.context.get("name")}`);

forEach({
  ref: this.context.ref("items"),
  generator: ({ ref: itemRef }) => text(itemRef.get().title), // ❌ то же для каждого item
});
```

После `textInput` или `set` пользователь на клиенте **не увидит** обновление, если текст собран из `get()`.

## Когда `get()` уместен

**Не** в props «показать/редактировать поле», а там, где нужно **решение на BFF** в момент вызова:

- **`onClick` / `onSubmit` / сервисы** — прочитать, посчитать, вызвать API; если ref на поле уже есть — **`ref.set(...)`**, иначе `context.set({ path: … })` (см. [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md))
- **Структура дерева** — разный набор блоков от длины списка, роли, feature-flag (тогда часто нужен `await context.init()` в `boot()` — скилл [context-init-in-boot](../context-init-in-boot/SKILL.md))
- **`boot()`** — нормализация данных до первой сериализации

```typescript
const greetingRef = this.context.ref("greeting");

onSubmit: () => {
  const name = this.context.get("name"); // ✅ логика submit, ref на name не обязателен
  greetingRef.set(`Привет, ${name}`); // ✅ запись через ref, если он уже создан
};
```

Для **видимости** блоков без пересборки всего дерева на сервере предпочитай **`when.*` на ref**, а не `if (this.context.get('loading'))` с разными статическими ветками, если достаточно показать/скрыть на клиенте.

## Чеклист агента

- [ ] Output/input/currency/datetime/image и т.п. — **`ref`**, не `get()` в аргументах
- [ ] `forEach` generator — **`itemRef.ref('…')`**, не `itemRef.get().…` для отображения
- [ ] Условия видимости — **`when.*(this.context.ref('…'), …)`**, где возможно
- [ ] `context.get()` / `ref.get()` в рендере только для **структуры** или с **`init()` в `boot()`**; осознанный trade-off «статичная ветка»
- [ ] Строки с контекстом — **`ref().toString()`**, не `get()`

## Связанные скиллы

- [context-ref-in-strings](../context-ref-in-strings/SKILL.md) — ref в шаблонных строках
- [strict-context-ref](../strict-context-ref/SKILL.md) — `ContextRef<C, P>` на странице
- [context-value-ref-in-api](../context-value-ref-in-api/SKILL.md) — `ContextValueRef` в параметрах хелперов
- [context-init-in-boot](../context-init-in-boot/SKILL.md) — когда рендер всё же читает `get()`
