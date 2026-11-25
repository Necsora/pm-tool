# Feature Specification: Project Entity with Tasks

**Feature Branch**: `001-project-entity`

**Created**: 2026-11-25

**Status**: Draft

**Input**: User description: "l'applicativo deve avere una entita progetto che consente di contenere delle entita di tipo task un progetto ha una data inizio una data fine un project manager una descrizione e un nome"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and view a project (Priority: P1)

A user creates a new project by providing its name, description, start date, end date and project manager. The project is saved and can be viewed in a list of projects and in a detail view showing all its attributes.

**Why this priority**: The project is the core container of the application. Without it, no other planning entity (tasks included) can exist.

**Independent Test**: Create a project with all attributes, then open the project list and the project detail: all entered values are shown correctly.

**Acceptance Scenarios**:

1. **Given** no projects exist, **When** a user creates a project with name, description, start date, end date and project manager, **Then** the project appears in the project list and its detail view shows all the entered values.
2. **Given** a project creation form, **When** the user omits the name, the start date, the end date or the project manager, **Then** the project is not created and the user is told which fields are missing.
3. **Given** a project creation form, **When** the user enters an end date earlier than the start date, **Then** the project is not created and the user is told the dates are inconsistent.

---

### User Story 2 - Add and manage tasks inside a project (Priority: P1)

A user opens a project and adds tasks to it. Each task belongs to exactly one project. The user can see all tasks of a project, edit them and remove them.

**Why this priority**: The ability of a project to contain tasks is the main purpose of the feature; together with Story 1 it forms the minimum viable product.

**Independent Test**: Open an existing project, add two tasks, edit one, delete the other, and verify the project's task list reflects each change.

**Acceptance Scenarios**:

1. **Given** an existing project, **When** a user adds a task to it, **Then** the task appears in that project's task list and in no other project's list.
2. **Given** a project with tasks, **When** a user opens the project, **Then** all of its tasks are listed.
3. **Given** a task in a project, **When** a user edits or deletes it, **Then** the project's task list reflects the change.

---

### User Story 3 - Edit and delete a project (Priority: P2)

A user updates a project's name, description, dates or project manager, or deletes a project that is no longer needed. Deleting a project also removes the tasks it contains, after the user confirms.

**Why this priority**: Projects change over time and mistakes must be fixable, but the feature is already useful without this.

**Independent Test**: Edit a project's attributes and verify they are updated; delete a project with tasks and verify both the project and its tasks are gone.

**Acceptance Scenarios**:

1. **Given** an existing project, **When** a user changes its attributes with valid values, **Then** the updated values are shown in the list and detail view.
2. **Given** a project containing tasks, **When** a user deletes it and confirms, **Then** the project and all its tasks are removed.
3. **Given** a project containing tasks, **When** a user starts deleting it and cancels the confirmation, **Then** nothing is removed.

---

### Edge Cases

- A project with start date equal to end date is valid (one-day project).
- Changing a project's dates so that existing tasks fall outside the new date range is allowed; the system does not delete or alter those tasks.
- Two projects may share the same name; each is identified uniquely by the system.
- A project with no tasks is valid and shows an empty task list.
- A name or description containing only whitespace is treated as empty.
- Very long names or descriptions are rejected with a clear message once a maximum length is exceeded.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to create a project with a name, a description, a start date, an end date and a project manager.
- **FR-002**: System MUST require name, start date, end date and project manager for every project; the description is optional.
- **FR-003**: System MUST reject a project whose end date is earlier than its start date.
- **FR-004**: System MUST show all projects in a list, including at least name, project manager, start date and end date.
- **FR-005**: System MUST show a project detail view with all its attributes and the tasks it contains.
- **FR-006**: System MUST allow users to edit all attributes of an existing project, applying the same validation as on creation.
- **FR-007**: System MUST allow users to delete a project, asking for confirmation first.
- **FR-008**: System MUST delete all tasks of a project when the project is deleted.
- **FR-009**: System MUST allow users to add tasks to an existing project.
- **FR-010**: System MUST ensure every task belongs to exactly one project and is never shown under another project.
- **FR-011**: System MUST allow users to edit and delete tasks of a project.
- **FR-012**: System MUST allow a project to exist with zero tasks and with an unlimited number of tasks.
- **FR-013**: System MUST persist projects and tasks so they are available in later sessions.
- **FR-014**: System MUST identify the project manager as a person known to the system, so that the same person can manage multiple projects.

### Key Entities *(include if feature involves data)*

- **Project**: A planned piece of work. Attributes: name, description, start date, end date, project manager. Contains zero or more tasks.
- **Task**: A unit of work that belongs to exactly one project. Its detailed attributes (title, status, assignee, dates, etc.) are out of scope for this feature beyond a minimal identifying title and are to be defined in a later feature.
- **Project Manager**: The person responsible for a project. Referenced by a project; one person can manage many projects.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can create a complete project (all fields) in under 1 minute.
- **SC-002**: A user can add a task to an existing project in under 30 seconds.
- **SC-003**: 100% of attempts to save a project with missing required fields or inconsistent dates are rejected with an explanatory message.
- **SC-004**: 100% of tasks displayed under a project belong to that project, and after deleting a project none of its tasks remain visible.
- **SC-005**: The project list and a project's task list display in under 2 seconds with up to 500 projects and 1,000 tasks per project.
- **SC-006**: 90% of first-time users complete "create a project and add a task" without assistance.

## Assumptions

- Dates are calendar dates (no time of day) and are interpreted without time-zone ambiguity.
- The project manager is chosen from the people known to the system; how people/users are created and authenticated is out of scope for this feature (a minimal list of people is assumed to be available).
- Detailed task attributes and task workflows (status, assignee, priority, dependencies) are out of scope and covered by a later feature; here a task needs only a title and its project.
- Access control and roles (who may create, edit or delete projects) are out of scope; all users of the application can perform every operation.
- Archiving or soft-deleting projects is out of scope; deletion is permanent after confirmation.
- Single-language user interface for v1; localisation is out of scope.
- All components used to deliver this feature comply with the project constitution (open-source only, AGPL-3.0-compatible).
