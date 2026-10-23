---
name: local-actions-boundaries
description: >-
  Defines the boundary between Matreshka LocalAction and server handlers. Use
  when adding or reviewing dialog, popover, button, form, or entry interactions,
  especially when replacing server component commands with local actions.
---

# Matreshka LocalAction и серверные обработчики

## Основное правило

`LocalAction` передаётся непосредственно в реакцию на событие и выполняется
клиентом. Не вызывай `component.local.*` или `instance.local.*` внутри тела
серверного обработчика.

Серверный обработчик — callback, который выполняется на BFF:

```ts
onClick: () => {
  // код выполняется на BFF
};
```

## Локальные действия

Если взаимодействие меняет только UI или вызывает client-only API, передавай
`LocalAction` напрямую. Это относится не только к `close`, но и к
`local.set(...)`, анимациям, clipboard и другим client-only действиям.

```ts
button(
  {
    onClick: [sidebarOpenRef.local.set(false), dialog.local.close()],
  },
  [text("Закрыть")],
);
```

Для одного DOM-инстанса используй `instance.local.*`, для всех активных
инстансов компонента — `component.local.*`.

Не вызывай local action внутри серверного callback:

```ts
onClick: () => {
  dialog.local.close();
};
```

## Граница между LocalAction и server logic

Если обработчик выполняет серверную работу — сохранение, очистку контекста,
проверку, навигацию или вызов доменной логики — эта работа остаётся в серверном
обработчике.

То же правило относится к любому локальному эффекту: `close`, `local.set(...)`,
анимации, clipboard и другим `LocalAction`.

LocalAction должен быть серверным по сценарию, если этот эффект является частью
бизнес-логики, то есть выполняется только в конкретном случае.

Используй серверную команду, если эффект является частью серверного
сценария:

```ts
onClick: () => {
  saveChanges();
  dialog.close();
};
```

Если же при нажатии на кнопку происходит И серверная логика И локальный эффект
независимо от логики, то `LocalAction` может быть локальным.

Если API поддерживает массив действий, серверную и локальную части объявляй
рядом, а не вкладывай local action в серверный callback:

```ts
onClick: [() => clearErrors(), dialog.local.close()];
```

Порядок действий должен соответствовать сценарию. Не закрывай entry до
обязательной серверной операции, если ошибка операции должна оставить entry
открытым. То же самое относится к любому другому локальному эффекту, который не
должен происходить при ошибке серверной операции.

## Проверка

- [ ] LocalAction находится непосредственно в свойстве события или массиве actions.
- [ ] Внутри server callback нет вызова `*.local.*`.
- [ ] LocalAction используется только когда локальный эффект не зависит от результата server logic.
- [ ] Server-сценарий управляет локальным эффектом, когда тот является частью бизнес-логики.
- [ ] Серверные операции и проверки остались серверными.
- [ ] Для dialog/popover выбран правильный target: component или instance.
- [ ] Проверены TypeScript и lint изменённых файлов.
