---
name: epf-erf-build
description: >
  Этот скилл MUST быть вызван когда пользователь просит собрать/скомпилировать внешнюю обработку (EPF) или внешний отчёт (ERF) из XML-исходников.
  SHOULD также вызывать после модификации XML-исходников обработки или отчёта.
  Do NOT использовать для разборки EPF/ERF — используй epf-erf-dump.
argument-hint: <Name>
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
---

# /epf-erf-build — Сборка обработки или отчёта

Собирает EPF или ERF файл из XML-исходников с помощью платформы 1С. Один скрипт работает для обоих форматов — формат определяется по расширению выходного файла (`.epf` / `.erf`).

## Usage

```
/epf-erf-build <Name> [SrcDir] [OutDir]
```

| Параметр | Обязательный | По умолчанию | Описание                          |
|----------|:------------:|--------------|-----------------------------------|
| Name     | да           | —            | Имя объекта (имя корневого XML)   |
| SrcDir   | нет          | `src`        | Каталог исходников                |
| OutDir   | нет          | `build`      | Каталог для результата            |

## Параметры подключения

Прочитай `.v8-project.json` из корня проекта. Возьми `v8path` (путь к платформе) и разреши базу для сборки:
1. Если пользователь указал параметры подключения (путь, сервер) — используй напрямую
2. Если указал базу по имени — ищи по id / alias / name в `.v8-project.json`
3. Если не указал — сопоставь текущую ветку Git с `databases[].branches`
4. Если ветка не совпала — используй `default`
5. Если `.v8-project.json` нет или баз нет — создай пустую ИБ в `./base`
Если `v8path` не задан — автоопределение: `Get-ChildItem "C:\Program Files\1cv8\*\bin\1cv8.exe" | Sort -Desc | Select -First 1`
Если использованная база не зарегистрирована — после выполнения предложи добавить через `/db-list add`.

## Команда

```powershell
powershell.exe -NoProfile -File .claude/skills/epf-erf-build/scripts/epf-build.ps1 <параметры>
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
| `-SourceFile <путь>` | да | Путь к корневому XML-файлу исходников |
| `-OutputFile <путь>` | да | Путь к выходному EPF/ERF-файлу |

> `*` — нужен либо `-InfoBasePath`, либо пара `-InfoBaseServer` + `-InfoBaseRef`

## Коды возврата

| Код | Описание        |
|-----|-----------------|
| 0   | Успешная сборка |
| 1   | Ошибка (см. лог)|

## Ссылочные типы

Если объект использует ссылочные типы конфигурации (`CatalogRef.XXX`, `DocumentRef.XXX`) — сборка в пустой базе упадёт с ошибкой XDTO. Зарегистрируй базу с целевой конфигурацией через `/db-list add`.

---

## EPF — Внешняя обработка

```powershell
# Сборка обработки (файловая база)
powershell.exe -NoProfile -File .claude/skills/epf-erf-build/scripts/epf-build.ps1 -InfoBasePath "C:\Bases\MyDB" -SourceFile "src\МояОбработка.xml" -OutputFile "build\МояОбработка.epf"

# Серверная база
powershell.exe -NoProfile -File .claude/skills/epf-erf-build/scripts/epf-build.ps1 -InfoBaseServer "srv01" -InfoBaseRef "MyDB" -UserName "Admin" -Password "secret" -SourceFile "src\МояОбработка.xml" -OutputFile "build\МояОбработка.epf"
```

---

## ERF — Внешний отчёт

```powershell
# Сборка отчёта (файловая база)
powershell.exe -NoProfile -File .claude/skills/epf-erf-build/scripts/epf-build.ps1 -InfoBasePath "C:\Bases\MyDB" -SourceFile "src\МойОтчёт.xml" -OutputFile "build\МойОтчёт.erf"

# Серверная база
powershell.exe -NoProfile -File .claude/skills/epf-erf-build/scripts/epf-build.ps1 -InfoBaseServer "srv01" -InfoBaseRef "MyDB" -UserName "Admin" -Password "secret" -SourceFile "src\МойОтчёт.xml" -OutputFile "build\МойОтчёт.erf"
```
