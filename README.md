# Multiagent Orchestrator MVP

Окремий проект для тестування PM-оркестратора і MVP-схеми мультиагентної системи.

## Що всередині

- `schemas/` — формальні JSON Schema для task, result, project state і PM output
- `docs/` — архітектурна специфікація і workflow blueprint

## MVP stack

- PM Agent
- Spec Agent
- Builder Agent
- QA Agent
- Shared State

## Структура

```text
multiagent-orchestrator-mvp/
  README.md
  docs/
    architecture-v1.1.md
    mvp-workflow-blueprint.md
  schemas/
    task.schema.json
    result.schema.json
    project-state.schema.json
    pm-output.schema.json
```

## Рекомендований запуск

1. Підключити `schemas/*.json` у валідатор.
2. Зібрати в `n8n` ланцюжок `PM -> Spec -> Builder -> QA`.
3. Зберігати `project_state` у Supabase або тимчасово в JSON storage.

## Перший тестовий сценарій

`Build an MVP auth flow with signup, login, and refresh tokens.`

## Статус

Проект готовий як стартова специфікація і набір артефактів для подальшого імпорту в workflow runtime.