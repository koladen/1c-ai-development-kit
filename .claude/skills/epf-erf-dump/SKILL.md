---
name: epf-erf-dump
description: >
  Этот скилл MUST быть вызван когда нужно разобрать EPF-файл обработки или ERF-файл отчёта 1С в XML-исходники.
  SHOULD также вызывать для изучения или модификации существующей обработки или отчёта.
  Do NOT использовать для сборки EPF/ERF — используй epf-erf-build.
argument-hint: <File>
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
---

# /epf-erf-dump — Разборка обработки или отчёта

Разбирает EPF или ERF файл во XML-исходники с помощью платформы 1С (иерархический формат). Один скрипт работает для обоих форматов — тип определяется автоматически.

## Usage

```
/epf-erf-dump <File> [OutDir]
```

| Параметр | Обязательный | По умолчанию | Описание                          |
|----------|:------------:|--------------|-----------------------------------|
| File     | да           | —            | Путь к EPF или ERF файлу          |
| OutDir   | нет          | `src`        | Каталог для выгрузки исходников   |

## Параметры подключения

Прочитай `.v8-project.json` из корня проекта. Возьми `v8path` (путь к платформе) и разреши базу:
1. Если пользователь указал параметры подключения (путь, сервер) — используй напрямую
2. Если указал базу по имени — ищи по id / alias / name в `.v8-project.json`
3. Если не указал — сопоставь текущую ветку Git с `databases[].branches`
4. Если ветка не совпала — используй `default`
5. Если `.v8-project.json` нет или баз нет — создай пустую ИБ в `./base`
Если `v8path` не задан — автоопределение: `Get-ChildItem "C:\Program Files\1cv8\*\bin\1cv8.exe" | Sort -Desc | Select -First 1`
Если использованная база не зарегистрирована — после выполнения предложи добавить через `/db-list add`.

## Команда

```powershell
powershell.exe -NoProfile -File .claude/skills/epf-erf-dump/scripts/epf-dump.ps1 <параметры>
```

### Параметры скрипта

| Параметр | Обязательный | Описание |
|----------|:------------:|----------|
| `-V8Path <путь>` | нет | Каталог bin платформы (или полный путь к 1cv8.exe) |
| `-InfoBasePath <путь>` | * | Файловая база |
| `-InfoBaseServer <сервер>` | * | Сервер 1С (для серверной базы) |
| `-InfoBaseRef <имя>` | * | Имя базы на сервере |
| `-UserName <имя>` | нет | Имя пользователя |
| `-Password <пароль>` | нет | Пароль |
| `-InputFile <путь>` | да | Путь к EPF/ERF-файлу |
| `-OutputDir <путь>` | да | Каталог для выгрузки исходников |
| `-Format <формат>` | нет | `Hierarchical` (по умолч.) / `Plain` |

> `*` — нужен либо `-InfoBasePath`, либо пара `-InfoBaseServer` + `-InfoBaseRef`

## Коды возврата

| Код | Описание          |
|-----|-------------------|
| 0   | Успешная разборка |
| 1   | Ошибка (см. лог)  |

## Формат `-Format Hierarchical`

Ключ `-Format Hierarchical` создаёт структуру каталогов:

```
<OutDir>/
├── <Name>.xml                    # Корневой файл
└── <Name>/
    ├── Ext/
    │   └── ObjectModule.bsl      # Модуль объекта (если есть)
    ├── Forms/
    │   ├── <FormName>.xml
    │   └── <FormName>/
    │       └── Ext/
    │           ├── Form.xml
    │           └── Form/
    │               └── Module.bsl
    └── Templates/
        ├── <TemplateName>.xml
        └── <TemplateName>/
            └── Ext/
                └── Template.<ext>
```

---

## EPF — Внешняя обработка

```powershell
# Разборка обработки (файловая база)
powershell.exe -NoProfile -File .claude/skills/epf-erf-dump/scripts/epf-dump.ps1 -InfoBasePath "C:\Bases\MyDB" -InputFile "build\МояОбработка.epf" -OutputDir "src"

# Серверная база
powershell.exe -NoProfile -File .claude/skills/epf-erf-dump/scripts/epf-dump.ps1 -InfoBaseServer "srv01" -InfoBaseRef "MyDB" -UserName "Admin" -Password "secret" -InputFile "build\МояОбработка.epf" -OutputDir "src"
```

---

## ERF — Внешний отчёт

```powershell
# Разборка отчёта (файловая база)
powershell.exe -NoProfile -File .claude/skills/epf-erf-dump/scripts/epf-dump.ps1 -InfoBasePath "C:\Bases\MyDB" -InputFile "build\МойОтчёт.erf" -OutputDir "src"

# Серверная база
powershell.exe -NoProfile -File .claude/skills/epf-erf-dump/scripts/epf-dump.ps1 -InfoBaseServer "srv01" -InfoBaseRef "MyDB" -UserName "Admin" -Password "secret" -InputFile "build\МойОтчёт.erf" -OutputDir "src"
```
