# Data Model: Project Entity with Tasks

## Person
| Field | Type | Rules |
|---|---|---|
| id | UUID | PK, system-generated |
| name | string(200) | required, non-blank |
| email | string(254) | required, unique, valid email format |

Deletion is rejected while the person is project manager of at least one project.

## Project
| Field | Type | Rules |
|---|---|---|
| id | UUID | PK, system-generated |
| name | string(200) | required, non-blank after trim; not unique (edge case) |
| description | string(5000) | optional |
| start_date | date | required |
| end_date | date | required; MUST be ≥ start_date (CHECK + API validation) |
| project_manager_id | UUID | required, FK → Person.id |
| created_at / updated_at | timestamp | system-managed |

## Task
| Field | Type | Rules |
|---|---|---|
| id | UUID | PK, system-generated |
| project_id | UUID | required, FK → Project.id, `ON DELETE CASCADE`; immutable after creation |
| title | string(200) | required, non-blank after trim |
| created_at / updated_at | timestamp | system-managed |

## Relationships
- Person 1 — 0..* Project (project manager)
- Project 1 — 0..* Task (a task belongs to exactly one project)

## Notes
- Changing project dates never modifies or deletes tasks (edge case).
- No state transitions in this feature.
