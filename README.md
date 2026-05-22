# Multiagent Orchestrator MVP

Окремий проект для тестування PM-оркестратора і MVP-схеми мультиагентної системи.

## Що всередині

- `schemas/` — формальні JSON Schema для task, result, project state і PM output
- `docs/` — архітектурна специфікація і workflow blueprint
- `prompts/` — готові системні prompt-файли для MVP-агентів
- `examples/` — еталонні JSON-приклади для PM, Spec, Builder, QA і project state
- `workflows/` — importable n8n workflow JSON для MVP skeleton

## Що вже закрито в протоколі

- `protocol_version` для безпечної еволюції схем
- `session_id` + `run_id` для групування одного orchestration cycle
- `retry_count` + `max_retries` для керованих повторних запусків
- `error` у Result Protocol для діагностики і retry-рішень
- явний `Error Handling` у PM prompt

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
  examples/
    pm-output.example.json
    spec-result.example.json
    builder-result.example.json
    qa-result.example.json
    project-state.example.json
  prompts/
    pm-agent.md
    spec-agent.md
    builder-agent.md
    qa-agent.md
  schemas/
    task.schema.json
    result.schema.json
    project-state.schema.json
    pm-output.schema.json
  workflows/
    multiagent-orchestrator-mvp.json
```

## Рекомендований запуск

1. Підключити `schemas/*.json` у валідатор.
2. Зібрати в `n8n` ланцюжок `PM -> Spec -> Builder -> QA`.
3. Зберігати `project_state` у Supabase або тимчасово в JSON storage.

## Перший тестовий сценарій

`Build an MVP auth flow with signup, login, and refresh tokens.`

## Що можна використати одразу

1. Імпортувати `workflows/multiagent-orchestrator-mvp.json` у `n8n` як self-contained skeleton.
2. Використати `examples/*.json` як golden samples для валідатора і тестів.
3. Замінити mock `Code` nodes на реальні LLM/API виклики без зміни контрактів.

## Статус

Проект готовий як стартова специфікація і набір артефактів для подальшого імпорту в workflow runtime.