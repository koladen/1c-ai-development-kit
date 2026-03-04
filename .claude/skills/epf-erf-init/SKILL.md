---
name: epf-erf-init
description: >
  Этот скилл MUST быть вызван когда пользователь просит создать новую пустую внешнюю обработку (EPF) или внешний отчёт (ERF) 1С (scaffold XML-исходников).
  SHOULD также вызывать при создании нового самостоятельного инструмента обработки или отчёта 1С.
  Do NOT использовать для добавления объекта в конфигурацию — используй meta-compile.
argument-hint: <Name> [--erf] [Synonym] [--with-skd]
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# /epf-erf-init — Создание новой обработки или отчёта

Генерирует минимальный набор XML-исходников для внешней обработки (EPF) или внешнего отчёта (ERF) 1С.

## Usage

```
/epf-erf-init <Name> [Synonym] [SrcDir]           # EPF (обработка)
/epf-erf-init <Name> [Synonym] [SrcDir] [--with-skd]  # ERF (отчёт)
```

---

## EPF — Внешняя обработка

### Параметры

| Параметр  | Обязательный | По умолчанию | Описание                            |
|-----------|:------------:|--------------|-------------------------------------|
| Name      | да           | —            | Имя обработки (латиница/кириллица)  |
| Synonym   | нет          | = Name       | Синоним (отображаемое имя)          |
| SrcDir    | нет          | `src`        | Каталог исходников относительно CWD |

### Команда

```powershell
powershell.exe -NoProfile -File .claude/skills/epf-erf-init/scripts/epf-init.ps1 -Name "<Name>" [-Synonym "<Synonym>"] [-SrcDir "<SrcDir>"]
```

### Что создаётся

```
<SrcDir>/
├── <Name>.xml          # Корневой файл метаданных (4 UUID)
└── <Name>/
    └── Ext/
        └── ObjectModule.bsl  # Модуль объекта с 3 регионами
```

- Корневой XML содержит `MetaDataObject/ExternalDataProcessor` с пустыми `DefaultForm` и `ChildObjects`
- ClassId фиксирован: `c3831ec8-d8d5-4f93-8a22-f9bfae07327f`
- Файл создаётся в UTF-8 с BOM

### Дальнейшие шаги (EPF)

- Добавить форму: `/epf-add-form`
- Добавить макет: `/template-add`
- Добавить справку: `/help-add`
- Собрать EPF: `/epf-erf-build`

---

## ERF — Внешний отчёт

### Параметры

| Параметр  | Обязательный | По умолчанию | Описание                              |
|-----------|:------------:|--------------|---------------------------------------|
| Name      | да           | —            | Имя отчёта (латиница/кириллица)       |
| Synonym   | нет          | = Name       | Синоним (отображаемое имя)            |
| SrcDir    | нет          | `src`        | Каталог исходников относительно CWD   |
| --WithSKD | нет          | —            | Создать пустую СКД и привязать к MainDataCompositionSchema |

### Команда

```powershell
powershell.exe -NoProfile -File .claude/skills/epf-erf-init/scripts/erf-init.ps1 -Name "<Name>" [-Synonym "<Synonym>"] [-SrcDir "<SrcDir>"] [-WithSKD]
```

### Что создаётся

```
<SrcDir>/
├── <Name>.xml          # Корневой файл метаданных (4 UUID)
└── <Name>/
    └── Ext/
        └── ObjectModule.bsl  # Модуль объекта с 3 регионами
```

При `--WithSKD` дополнительно:

```
<SrcDir>/<Name>/
    Templates/
    ├── ОсновнаяСхемаКомпоновкиДанных.xml        # Метаданные макета
    └── ОсновнаяСхемаКомпоновкиДанных/
        └── Ext/
            └── Template.xml                      # Пустая СКД
```

- Корневой XML содержит `MetaDataObject/ExternalReport` с пустыми `DefaultForm`, `MainDataCompositionSchema` и `ChildObjects`
- При `--WithSKD` — `MainDataCompositionSchema` заполняется ссылкой на макет, `ChildObjects` содержит `<Template>`
- ClassId фиксирован: `e41aff26-25cf-4bb6-b6c1-3f478a75f374`
- Файл создаётся в UTF-8 с BOM

### Дальнейшие шаги (ERF)

- Добавить форму: `/form-add`
- Добавить макет: `/template-add`
- Добавить справку: `/help-add`
- Собрать ERF: `/epf-erf-build`
