---
name: interface-edit
description: >
  Этот скилл MUST быть вызван когда нужно настроить командный интерфейс подсистемы 1С: скрыть или показать команды, разместить в группах, настроить порядок.
  SHOULD также вызывать после добавления объектов в подсистему для настройки их видимости.
  Do NOT использовать для создания подсистемы — используй subsystem-compile; для управления составом — используй subsystem-edit.
argument-hint: <CIPath> <Operation> <Value>
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
---

# /interface-edit — редактирование CommandInterface.xml

Операции: hide, show, place, order, subsystem-order, group-order. Подробнее: `.claude/skills/interface-edit/reference.md`

```powershell
powershell.exe -NoProfile -File '.claude/skills/interface-edit/scripts/interface-edit.ps1' -CIPath '<path>' -Operation hide -Value '<cmd>'
```
