# Implementation Plan: Project Entity with Tasks

**Branch**: `001-project-entity` | **Date**: 2026-11-25 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-project-entity/spec.md`

## Summary

Introduce the **Project** entity (name, description, start date, end date, project manager) that contains **Task** entities, with full create/read/update/delete for both, plus a minimal **Person** entity so a project manager can be referenced. Approach: a web application with a REST backend (FastAPI + SQLAlchemy + PostgreSQL) and a single-page frontend (React + Vite). Deleting a project cascades to its tasks. All stack choices are OSI-approved and AGPL-3.0-compatible (see research.md).

## Technical Context

**Language/Version**: Python 3.12 (backend), TypeScript 5 (frontend)

**Primary Dependencies**: FastAPI, Pydantic v2, SQLAlchemy 2.x, Alembic, React 18, Vite

**Storage**: PostgreSQL 16 (SQLite in-memory permitted for fast unit tests)

**Testing**: pytest + httpx (backend contract/integration/unit), Vitest + Testing Library (frontend)

**Target Platform**: Linux server (containerised), modern evergreen browsers

**Project Type**: web application (backend + frontend)

**Performance Goals**: project list and task list render in < 2 s with 500 projects / 1,000 tasks per project (SC-005)

**Constraints**: open-source-only dependencies (constitution); date-only fields without time-zone ambiguity; no authentication in this feature

**Scale/Scope**: single organisation, ~500 projects, ~1,000 tasks per project, 3 entities, ~4 screens

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle / Constraint | Status | Notes |
|---|---|---|
| I. Open source only; own code AGPL-3.0 | PASS | `LICENSE` already at repo root; no proprietary components planned |
| Every dependency OSI-approved & AGPL-compatible, license recorded | PASS | Licenses recorded in research.md (MIT, BSD, Apache-2.0, PostgreSQL License) |
| No hard dependency on proprietary hosted services | PASS | Self-hosted PostgreSQL; no SaaS |
| Source offered to network users (AGPL) | PASS (deferred) | A "Source code" link in the UI footer is part of the frontend shell tasks |

**Post-design re-check**: PASS. Data model and contracts introduce no new dependencies.

## Project Structure

### Documentation (this feature)

```text
specs/001-project-entity/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── openapi.yaml
└── tasks.md             # created later by /speckit-tasks
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── models/          # Person, Project, Task (SQLAlchemy)
│   ├── schemas/         # Pydantic request/response models
│   ├── services/        # project_service, task_service, person_service
│   └── api/             # routers: people, projects, tasks
├── migrations/          # Alembic
└── tests/
    ├── contract/
    ├── integration/
    └── unit/

frontend/
├── src/
│   ├── components/      # ProjectForm, TaskList, ConfirmDialog
│   ├── pages/           # ProjectList, ProjectDetail
│   └── services/        # API client
└── tests/
```

**Structure Decision**: Web application layout (backend + frontend), since the product is a browser-based project management tool with a REST contract between the two.

## Complexity Tracking

No constitution violations; section not applicable.
