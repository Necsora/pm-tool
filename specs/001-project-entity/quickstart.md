# Quickstart: Validating Project Entity with Tasks

Prerequisites: Python 3.12, Node 20+, PostgreSQL 16 (or Docker). Commands assume the layout in [plan.md](plan.md); exact scripts are created during implementation.

1. **Start dependencies and apply migrations**: start PostgreSQL, then run the backend migrations (`alembic upgrade head` in `backend/`).
2. **Run backend and frontend**: start the API (`uvicorn`) and the frontend dev server (`npm run dev`).
3. **Run automated tests**: `pytest` in `backend/` and `npm test` in `frontend/`. Contract tests check behaviour against [contracts/openapi.yaml](contracts/openapi.yaml).

## Manual end-to-end scenarios

| # | Scenario | Expected |
|---|---|---|
| 1 | Create a person, then a project with all fields (Story 1) | Project in list; detail shows all values (SC-001) |
| 2 | Submit project with missing name / dates / manager | Rejected with per-field messages (FR-002) |
| 3 | Submit end date before start date | Rejected; same-day dates accepted (FR-003, edge case) |
| 4 | Add two tasks to a project, edit one, delete the other (Story 2) | Task list reflects each change; tasks absent from other projects (FR-010) |
| 5 | Edit project dates so tasks fall outside range | Saved; tasks untouched |
| 6 | Delete project, cancel confirmation, then delete and confirm (Story 3) | First: nothing removed; second: project and tasks gone (FR-008) |
| 7 | Seed 500 projects × 1,000 tasks, open list and a detail page | Each renders in < 2 s (SC-005) |

Data details: [data-model.md](data-model.md).
