# Research: Project Entity with Tasks

All Technical Context items were unspecified by the user; the decisions below resolve them. Every dependency is OSI-approved and AGPL-3.0-compatible (constitution).

| Decision | Choice | License | Rationale | Alternatives considered |
|---|---|---|---|---|
| Backend language/framework | Python 3.12 + FastAPI | PSF / MIT | Automatic OpenAPI contract, validation via Pydantic, simple CRUD | Django (heavier, fine too), Node/Express (less built-in validation) |
| Validation | Pydantic v2 | MIT | Enforces required fields and end ≥ start date at the boundary | Hand-written checks |
| ORM / migrations | SQLAlchemy 2.x + Alembic | MIT | Mature, supports cascade deletes and constraints | Raw SQL, Django ORM |
| Database | PostgreSQL 16 | PostgreSQL License (OSI) | Real FK/CHECK constraints, scales to target volume | SQLite (kept for fast tests only), MariaDB |
| Frontend | React 18 + Vite + TypeScript | MIT | Common, simple SPA for list/detail/forms | Server-rendered templates (less interactive), Vue |
| Backend tests | pytest + httpx | MIT / BSD | Contract tests directly against the OpenAPI behaviour | unittest |
| Frontend tests | Vitest + Testing Library | MIT | Native Vite integration | Jest |
| Project manager modelling | Separate `Person` entity referenced by `Project` | n/a | FR-014: same person manages many projects; auth out of scope so Person is a minimal record (name, email) | Free-text name (violates FR-014), full User/auth (out of scope) |
| Delete behaviour | DB-level `ON DELETE CASCADE` from Project to Task; Person delete blocked while managing a project | n/a | FR-008; avoids orphan projects | Soft delete (out of scope) |
| Dates | `DATE` type, ISO-8601 (`YYYY-MM-DD`) | n/a | No time-zone ambiguity (assumption) | Timestamps |
| Limits | name ≤ 200 chars, description ≤ 5,000, task title ≤ 200 | n/a | Edge case "very long names rejected" | Unlimited |
| Task scope | Task has only `title` + project link | n/a | Spec defers richer task attributes | Full task model now |

No NEEDS CLARIFICATION items remain.
