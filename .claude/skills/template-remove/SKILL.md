---
name: template-remove
description: >
  Этот скилл MUST быть вызван когда пользователь просит удалить/убрать макет из объекта 1С.
  SHOULD также вызывать при очистке неиспользуемых макетов.
  Do NOT использовать для добавления макета — используй template-add; для модификации содержимого — используй mxl-compile.
argument-hint: <ObjectName> <TemplateName>
disable-model-invocation: true
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# /template-remove — Удаление макета

Удаляет макет и убирает его регистрацию из корневого XML объекта.

## Usage

```
/template-remove <ObjectName> <TemplateName>
```

| Параметр     | Обязательный | По умолчанию | Описание                            |
|--------------|:------------:|--------------|-------------------------------------|
| ObjectName   | да           | —            | Имя объекта                         |
| TemplateName | да           | —            | Имя макета для удаления             |
| SrcDir       | нет          | `src`        | Каталог исходников                  |

## Команда

```powershell
powershell.exe -NoProfile -File .claude/skills/template-remove/scripts/remove-template.ps1 -ObjectName "<ObjectName>" -TemplateName "<TemplateName>" [-SrcDir "<SrcDir>"]
```

## Что удаляется

```
<SrcDir>/<ObjectName>/Templates/<TemplateName>.xml     # Метаданные макета
<SrcDir>/<ObjectName>/Templates/<TemplateName>/         # Каталог макета (рекурсивно)
```

## Что модифицируется

- `<SrcDir>/<ObjectName>.xml` — убирается `<Template>` из `ChildObjects`
- Для ExternalReport/Report: если удалённый макет был указан в `MainDataCompositionSchema` — значение очищается
