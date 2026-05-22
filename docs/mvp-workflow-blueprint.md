# MVP Multiagent Workflow Blueprint

This file describes the smallest useful n8n workflow for the audited multiagent architecture.

## Goal

Run a single request through:

1. PM planning
2. Spec generation
3. Builder execution
4. QA verification
5. Shared state update

## Recommended Files

- `schemas/task.schema.json`
- `schemas/result.schema.json`
- `schemas/project-state.schema.json`
- `schemas/pm-output.schema.json`

## Workflow Shape

### 1. Trigger

- Node type: `n8n-nodes-base.manualTrigger`
- Purpose: start a test run manually.

### 2. Build Initial Request

- Node type: `n8n-nodes-base.code`
- Purpose: normalize raw user input into a project envelope.
- Output example:

```json
{
  "project_id": "PROJ-001",
  "run_id": "RUN-2026-05-22-001",
  "user_request": "Build an MVP auth flow with signup, login, and refresh tokens",
  "state_version": 1
}
```

### 3. PM Planner

- Node type: `n8n-nodes-base.httpRequest` or `n8n-nodes-base.code`
- Purpose: call the PM model and force output in `pm-output.schema.json` format.
- Notes:
  - In production, use `response_format` or strict JSON instructions.
  - Validate PM output before dispatching any task.

### 4. Validate PM Output

- Node type: `n8n-nodes-base.code`
- Purpose: validate PM response against `pm-output.schema.json`.
- Failure path: stop run and return validation error.

### 5. Guard: Any Tasks?

- Node type: `n8n-nodes-base.if`
- Condition: `{{ $json.tasks_to_dispatch.length > 0 }}`
- False path: finish run with PM final user response.

### 6. Dispatch First Task

- Node type: `n8n-nodes-base.code`
- Purpose: select `tasks_to_dispatch[0]` and expose it as the active task.

### 7. Agent Router

- Node type: `n8n-nodes-base.if`
- Pattern: chain of `if` nodes for MVP agents.

Suggested order:

1. `if active_task.agent == "spec_agent"`
2. `if active_task.agent == "builder_agent"`
3. `if active_task.agent == "qa_agent"`

### 8. Agent Execution Nodes

- Node type: `n8n-nodes-base.httpRequest` or `n8n-nodes-base.code`
- One node per agent in MVP.
- Each node must return `result.schema.json` format.

Minimum agents:

1. `Spec Agent`
2. `Builder Agent`
3. `QA Agent`

### 9. Validate Agent Result

- Node type: `n8n-nodes-base.code`
- Purpose: validate the returned agent payload against `result.schema.json`.

### 10. Update Shared State

- Node type: `n8n-nodes-base.code`, `n8n-nodes-base.httpRequest`, or DB node
- Purpose:
  - increment `state_version`
  - attach new artifact IDs
  - move task IDs between active/completed
  - append decisions and risks

Target shape: `project-state.schema.json`

### 11. Loop Back to PM

- Node type: `n8n-nodes-base.code` or `n8n-nodes-base.httpRequest`
- Purpose: pass updated state plus last result back to PM.
- PM decides:
  - next task
  - clarification request
  - run completion

## Minimal Run Logic

```text
Manual Trigger
  -> Build Initial Request
  -> PM Planner
  -> Validate PM Output
  -> Guard Any Tasks?
    -> no: finish
    -> yes: Dispatch First Task
         -> Agent Router
         -> Agent Execute
         -> Validate Agent Result
         -> Update Shared State
         -> PM Planner
```

## Environment Variables

Recommended variables for n8n:

- `OPENROUTER_API_KEY`
- `PM_MODEL`
- `SPEC_MODEL`
- `BUILDER_MODEL`
- `QA_MODEL`
- `STATE_STORE_MODE`
- `STATE_STORE_URL`

## First MVP Constraint

Do not implement parallel execution on the first pass. Run one active task at a time until:

1. schemas validate reliably,
2. state updates are consistent,
3. PM routing produces stable next steps.

## Success Criteria

The blueprint is working when a single request can:

1. produce valid PM output,
2. dispatch one valid task,
3. receive one valid agent result,
4. update valid project state,
5. return to PM without losing context.