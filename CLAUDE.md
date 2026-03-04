# 1C AI Workspace — Claude Code Configuration

## О проекте

Портативная AI-конфигурация для разработки на 1С:Предприятие 8.3. Содержит 74 skills, документацию по XML форматам, JSON DSL спецификации, интеграцию с MCP серверами.

## Структура

```
.claude/skills/    — 74 skills для Claude Code (SKILL.md + скрипты)
.claude/docs/      — спецификации и гайды по форматам 1С
openspec/          — Specification-Driven Development
scripts/           — инфраструктурные скрипты (не hooks)
```

## MANDATORY: MCP-First Rule

**These rules apply to initialized 1C projects (where MCP tools are connected via `/1c-project-init`).**
**This workspace itself has only `rlm-toolkit` (global) and `playwright` (local).**

### When to use which MCP tool (in 1C projects)

| Situation | Tool | Action |
|-----------|------|--------|
| Need 1C platform syntax / method docs | `1c-help` | Call `helpsearch` BEFORE writing code |
| Working with BSP subsystems | `1c-ssl` | Call `ssl_search` to find correct BSP patterns |
| Need a code template / pattern | `1c-templates` | Call `template_search` BEFORE writing from scratch |
| Wrote BSL code | `1c-syntax-checker` | Call `check_syntax` ALWAYS after writing BSL |
| Complex logic / architecture review | `1c-code-checker` | Call to verify logic via 1C Companion |
| Working with managed forms XML | `1c-forms` | Call `get_form_schema` for structure reference |
| Need to remember project context | `rlm-toolkit` | Call `rlm_route_context` at session start |
| Work with 1C in browser (forms, data, testing) | `playwright` | Use `/1c-web-session` skill |

### Non-negotiable rules (in 1C projects)

- **NEVER** write 1C code without first checking `1c-help` for syntax
- **NEVER** use a BSP subsystem without checking `1c-ssl` for correct pattern
- **NEVER** skip `1c-syntax-checker` after writing BSL code
- **NEVER** guess a code template — search `1c-templates` first
- If MCP server is unavailable — say so explicitly, don't silently fall back

### MCP vs Grep decision (in 1C projects)

| Task | Use |
|------|-----|
| Search project source code | Grep/Glob |
| Search 1C metadata XML | Grep/Glob |
| 1C platform docs / syntax | `1c-help` |
| BSP patterns | `1c-ssl` |
| Code templates | `1c-templates` |

---

## MANDATORY: Skills-First Rule

**BEFORE writing any script, code, or solution — check if a skill exists.**

Workflow:
1. User asks for something → scan skill list below
2. Skill exists → use `Skill` tool immediately, do NOT reinvent
3. No skill → only then write custom code

This applies to ALL 1C operations: creating bases, loading configs, compiling objects, working with forms, BSP, SKD, roles, etc. **Never generate your own PowerShell/BAT scripts for operations that skills cover.**

## MANDATORY: Autonomy After Approval

**One approval point per task. After user says "ok" — execute autonomously.**

- Ask clarifying questions BEFORE showing the plan
- Show design+plan → get ONE approval
- After "ok": do NOT ask "can I proceed with step N?", "is this part ok?", "should I continue?"
- Only stop for blockers (impossible to continue, contradictory requirements)
- Report results at the end

## Task Routing (automatic)

AI determines the mode based on task complexity:

| Complexity | Mode | What to do |
|-----------|------|-----------|
| 1-2 objects, obvious | **direct** | Use skills directly, no ceremony |
| 3-5 tasks, needs design | **standard** | `/brainstorm` → brief plan → execute |
| 6+ tasks, architectural | **full** | `/brainstorm` → `/write-plan` → `/subagent-dev` |
| 6+ files, parallel work needed | **team** | "новая задача" → Agent Teams (Opus leader + Sonnet teammates). See `~/.claude/CLAUDE.md` for protocol |
| Formal spec management | **openspec** | `/openspec-proposal` → `/openspec-apply` |

## Skills (ключевые команды)

### Объекты метаданных
- `/meta-compile`, `/meta-edit`, `/meta-remove`, `/meta-validate` — CRUD для 23 типов объектов
- `/inspect` — анализ структуры объекта (реквизиты, ТЧ, формы, движения, типы)

### Формы
- `/form-compile`, `/form-edit`, `/form-add`, `/form-validate`, `/form-patterns`
- `/inspect` — анализ структуры формы (Form.xml: элементы, реквизиты, команды)
- `/help-add` — встроенная справка к объекту 1С

### Обработки и отчёты
- `/epf-init`, `/epf-build`, `/epf-dump`, `/epf-validate`, `/epf-add-form`
- `/erf-init`, `/erf-build`, `/erf-dump`, `/erf-validate`

### БСП интеграция
- `/epf-bsp-init` — регистрация в БСП
- `/epf-bsp-add-command` — добавление команды
- `/bsp-patterns` — паттерны работы с подсистемами

### СКД (отчёты)
- `/skd-compile`, `/skd-edit`, `/skd-validate`
- `/inspect` — анализ структуры СКД (наборы, поля, параметры, варианты, трассировка)

### Макеты (печатные формы)
- `/mxl-compile`, `/mxl-decompile`, `/mxl-validate`
- `/inspect` — анализ структуры MXL-макета (области, параметры, наборы колонок)
- `/template-add`, `/template-remove` — добавить/удалить макет к объекту конфигурации
- `/img-grid` — наложить сетку на изображение для определения пропорций колонок

### Роли и права
- `/role-compile`, `/role-validate`
- `/inspect` — аудит прав роли (Rights.xml: объекты, действия, RLS, шаблоны)

### Конфигурация и расширения
- `/cf-init`, `/cf-edit`, `/cf-validate`
- `/inspect` — обзор структуры конфигурации (объекты по типам, свойства)
- `/cfe-init`, `/cfe-borrow`, `/cfe-patch-method`, `/cfe-validate`, `/cfe-diff`

### Подсистемы
- `/subsystem-compile`, `/subsystem-edit`, `/subsystem-validate`
- `/inspect` — анализ структуры подсистемы (состав, CI, дерево иерархии)
- `/interface-edit`, `/interface-validate`

### База данных
- `/db-create`, `/db-list`, `/db-dump-cf`, `/db-load-cf`
- `/db-dump-xml`, `/db-load-xml`, `/db-update`, `/db-run`
- `/db-load-git` — умная загрузка изменений из Git

### Веб-клиент (Playwright)
- `/1c-web-session` — управление 1С в браузере: сеансы, навигация, формы, справочники, документы, тестовые данные

### Веб-публикация (Apache)
- `/web-publish` — публикация базы через portable Apache (генерирует default.vrd + httpd.conf, скачивает Apache при необходимости)
- `/web-unpublish` — удаление публикации (одной или всех)
- `/web-info` — статус Apache + список опубликованных баз
- `/web-stop` — остановка Apache (публикации сохраняются)
- `/web-test` — Playwright-автоматизация веб-клиента 1С (Node.js: autonomous/interactive/piped режимы, video recording, dom.mjs)

### Инициализация и тестирование
- `/1c-project-init` — инициализация/обогащение 1С проекта (skills, docs, CLAUDE.md, MCP)
- `/1c-test-runner` — AI-тестирование бизнес-логики через `1c-ai-debug` MCP (без внешних зависимостей)
- `/playwright-test` — scaffold UI-теста после деплоя (package.json + spec.js с JSONL-логом)

### Workflow
- `/brainstorm` — **основной**: обсуждение → план → автономное выполнение (express/standard/full)
- `/write-plan` — отдельно создать tasks.md из design.md (обычно вызывается из brainstorm)
- `/subagent-dev` — отдельно выполнить tasks.md субагентами (обычно вызывается из brainstorm full)
- `/1c-help-mcp` — поиск по документации платформы
- `/1c-query-opt` — оптимизация запросов

### OpenSpec
- `/openspec-proposal` — создать предложение изменения
- `/openspec-apply` — реализовать одобренное изменение
- `/openspec-archive` — архивировать завершённое изменение

## Правила разработки

### 1С кодирование
- Следовать стандартам БСП и ITS
- Кириллица для кода 1С (BSL), латиница для инфраструктуры
- Табы для отступов в BSL коде
- UTF-8 BOM для PowerShell скриптов с кириллицей

### Workflow доработок
- Для любых доработок: `/brainstorm` (сам выберет режим express/standard/full)
- Для формальных спецификаций: `/openspec-proposal` → `/openspec-apply`
- Одно одобрение → автономная реализация → отчёт в конце
- НИКОГДА не спрашивать разрешения после одобрения плана

### Git безопасность
- НИКОГДА force push на main/master
- НЕ коммитить .env, credentials, ключи
- НЕ пропускать hooks без явного запроса
- Предупреждать перед деструктивными операциями

### Выбор модели
- Sonnet для 90%+ задач (генерация, ревью, вопросы)
- Opus только для критических: архитектура, безопасность, production баги

### Контекст
- RLM-first: проверяй RLM перед чтением файлов
- Сохраняй решения в RLM после завершения задач
- Task agents для параллельных задач (изоляция контекста)

## MCP серверы

### В этом проекте (1c-AI-workspace)

Это workspace/toolkit — не 1С проект. Подключены только:
- `rlm-toolkit` — глобальный (`~/.claude/mcp.json`), персистентная память
- `playwright` — локальный, управление браузером для тестирования

### При инициализации 1С проекта (`/1c-project-init`)

Шаблон `.mcp.json` (`.claude/skills/1c-project-init/templates/mcp.json.template`) разворачивает:

**Общие 1С (CT103, YOUR_MCP_SERVER):**
- `1c-help` (:8003) — документация платформы (`helpsearch`)
- `1c-ssl` (:8008) — паттерны БСП (`ssl_search`)
- `1c-templates` (:8004) — шаблоны кода (`template_search`)
- `1c-syntax-checker` (:8002) — проверка синтаксиса BSL
- `1c-code-checker` (:8007) — проверка логики через 1С:Напарник
- `1c-forms` (:8011) — схема управляемых форм (`get_form_schema`)

**Глобальные:**
- `rlm-toolkit` (CT105, YOUR_RLM_SERVER:8200) — персистентная память

**Проектные (настраиваются под конкретный проект):**
- `mcp-bsl-lsp` (CT100) — LSP-анализ BSL через Docker-контейнер проекта
- `1c-ai-debug` — MCP-мост к HTTP-сервису 1С (запросы, метаданные, данные)

**Локальные:**
- `playwright` — управление браузером (веб-клиент 1С, тестирование)

## OpenSpec

Методология Specification-Driven Development:
- `openspec/project.md` — контекст и соглашения проекта
- `openspec/changes/` — активные предложения изменений
- `openspec/specs/` — текущие спецификации возможностей

Skills: `/openspec-proposal`, `/openspec-apply`, `/openspec-archive`

## Инфраструктура

См. полную карту в `~/.claude/CLAUDE.md`. Ключевое для этого проекта:
- CT103 (mcp-common): YOUR_MCP_SERVER — 6 общих MCP серверов, разворачиваются в проекты через `/1c-project-init`
- CT105 (rlm): YOUR_RLM_SERVER — RLM-toolkit:8200 (глобальный)
- CT107 (onec-dev): YOUR_EDT_SERVER — Docker: onec-server-24/25/27, onec-postgres, onec-web-24/25
