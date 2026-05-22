# Мультиагентна архітектура | v1.1

## 1. Принципи системи

Це не просто набір промптів, а операційна специфікація для PM-оркестратора і субагентів.

### 1.1 Базові правила

1. Усі субагенти працюють тільки через PM Agent.
2. Усі задачі, результати й артефакти проходять через єдиний JSON-протокол.
3. Єдине джерело правди для проєкту — shared state.
4. Кожен агент відповідає лише за свою зону компетенції.
5. Якщо бракує критичного контексту — агент повертає `needs_clarification`, а не вигадує.

### 1.2 Source of Truth

У системі має існувати окремий шар збереження стану:
- `project_state`
- `tasks`
- `artifacts`
- `decisions`
- `execution_runs`

Це може бути `Supabase`, `PostgreSQL`, JSON-таблиця або навіть файлова структура для MVP. Важливо не де зберігати, а щоб усі агенти працювали з однією версією стану.

---

## 2. JSON Task Protocol

Єдиний формат обміну між PM-оркестратором і всіма субагентами.

### 2.1 Задача від PM → Субагент

```json
{
  "project_id": "PROJ-001",
  "run_id": "RUN-2026-05-22-001",
  "task_id": "TASK-001",
  "parent_task_id": null,
  "step_id": "STEP-003",
  "state_version": 4,
  "agent": "backend_agent",
  "priority": "high",
  "status": "pending",
  "title": "Створити REST API для авторизації",
  "description": "Реалізувати ендпоінти /login, /register, /refresh-token з JWT",
  "depends_on": ["TASK-000"],
  "input": {
    "requirements": ["JWT auth", "refresh tokens", "rate limiting"],
    "context": {
      "stack": ["Node.js", "Express", "PostgreSQL"],
      "environment": "staging",
      "business_goal": "безпечна email/password авторизація"
    },
    "input_artifacts": [
      {
        "artifact_id": "ART-REQ-001",
        "type": "spec",
        "format": "application/json",
        "name": "auth-feature-spec.json",
        "content": {
          "auth_method": "jwt",
          "token_rotation": true
        }
      }
    ]
  },
  "expected_output": {
    "artifact_types": ["code", "api_spec", "dependency_list"],
    "acceptance_criteria": [
      "Ендпоінти повертають JWT access + refresh токени",
      "Rate limit 5 req/min на /login",
      "Валідація через Zod-схеми",
      "Є OpenAPI або JSON API contract"
    ]
  },
  "constraints": {
    "max_tokens": 4096,
    "timeout_sec": 120,
    "do_not": [
      "Не використовувати Passport.js",
      "Не зберігати паролі у відкритому вигляді"
    ]
  },
  "created_at": "2026-05-22T10:30:00+02:00"
}
```

### 2.2 Результат від Субагент → PM

```json
{
  "project_id": "PROJ-001",
  "run_id": "RUN-2026-05-22-001",
  "task_id": "TASK-001",
  "agent": "backend_agent",
  "status": "completed",
  "summary": "Підготовлено API-контракт і 3 серверні файли для auth flow.",
  "artifacts": [
    {
      "artifact_id": "ART-API-001",
      "type": "api_spec",
      "name": "auth.openapi.json",
      "format": "application/json",
      "content": {
        "openapi": "3.1.0"
      }
    },
    {
      "artifact_id": "ART-CODE-001",
      "type": "code",
      "name": "auth.routes.ts",
      "format": "text/typescript",
      "content": "// ... код ..."
    },
    {
      "artifact_id": "ART-DEP-001",
      "type": "dependency_list",
      "name": "backend-deps.json",
      "format": "application/json",
      "content": {
        "dependencies": [
          {"name": "zod", "version": "^3.23.8"},
          {"name": "jsonwebtoken", "version": "^9.0.2"}
        ]
      }
    }
  ],
  "issues": [],
  "decisions": [
    "Access token TTL 15 хв",
    "Refresh token rotation увімкнено"
  ],
  "next_actions": [
    "Передати api_spec у frontend_agent",
    "Передати feature scope у qa_manual_agent"
  ],
  "metrics": {
    "tokens_used": 2847,
    "execution_time_sec": 34
  },
  "completed_at": "2026-05-22T10:31:14+02:00"
}
```

### 2.3 Статуси задач

| Статус | Значення |
|--------|----------|
| `pending` | Задача створена, чекає виконання |
| `in_progress` | Субагент працює |
| `completed` | Виконано успішно |
| `failed` | Помилка виконання |
| `blocked` | Задача блокується залежністю |
| `review` | PM перевіряє результат |
| `needs_clarification` | Немає критично важливого контексту |

### 2.4 Типи артефактів

Універсальний перелік артефактів, який покриває більшість агентів:
- `spec`
- `decision_log`
- `api_spec`
- `schema`
- `code`
- `ui_wireframe`
- `copy`
- `test_cases`
- `bug_report`
- `deployment_plan`
- `dependency_list`
- `runbook`

---

## 3. Shared State Schema

PM має читати і оновлювати shared state після кожного завершеного кроку.

```json
{
  "project_id": "PROJ-001",
  "project_name": "Auth Service MVP",
  "current_stage": "implementation",
  "state_version": 4,
  "source_of_truth": {
    "requirements_artifact_id": "ART-REQ-001",
    "architecture_artifact_id": "ART-ARCH-001",
    "latest_api_spec_artifact_id": "ART-API-001"
  },
  "active_tasks": ["TASK-001", "TASK-002"],
  "completed_tasks": ["TASK-000"],
  "decisions": [
    {
      "id": "DEC-001",
      "summary": "Обрано JWT auth замість session-based auth",
      "owner": "architect_agent"
    }
  ],
  "artifacts": ["ART-REQ-001", "ART-ARCH-001", "ART-API-001"],
  "risks": [
    "Frontend не можна запускати до появи api_spec або approved mocks"
  ],
  "updated_at": "2026-05-22T10:31:20+02:00"
}
```

---

## 4. PM Prompt, MVP Registry, PM Output і схемы винесені в окремі schema/doc файли цього проекту.