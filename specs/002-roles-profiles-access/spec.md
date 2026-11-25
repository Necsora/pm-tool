# Feature Specification: Roles, Profiles, Groups and Access Control

**Feature Branch**: `002-roles-profiles-access`

**Created**: 2026-11-25

**Status**: Draft

**Input**: User description: "nell'applicativo ci devono essere ruoli, profili e gruppi. Un profilo e' un insieme di ruoli e un gruppo e' un insieme di utenti. Ai gruppi possono essere associati i profili. Gli utenti hanno i ruoli che sono la somma dei gruppi a cui appartengono; ruoli e profili non possono essere dati ai singoli utenti. Ogni ruolo abilita una cosa che puo essere un menu o un bottone ecc. Ci sono due gruppi di default: amministratori e utenti. L'amministratore puo inserire ruoli, associare ai profili i ruoli, gestire i gruppi, aggiungere gli utenti."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access adapts to the user's groups (Priority: P1)

A role enables one specific element of the application (a menu entry, a button, an action, and so on). A profile is a set of roles. A group is a set of users, and profiles are associated to groups. A user's roles are the sum of the roles of all profiles of all groups the user belongs to. The user sees only the menus and controls enabled by those roles and can perform only the corresponding actions.

**Why this priority**: This is the core value of the feature: without enforcement of roles on what users see and do, groups, profiles and roles have no effect.

**Independent Test**: Create two groups with different profiles, put one user in each, sign in as each and verify menus, buttons and actions differ exactly according to the roles reached through the groups.

**Acceptance Scenarios**:

1. **Given** a user in a group whose profile includes the role enabling the "Projects" menu, **When** the user opens the application, **Then** the "Projects" menu entry is visible and usable.
2. **Given** a user none of whose groups reaches the role enabling the "Projects" menu, **When** the user opens the application, **Then** that menu entry is not shown.
3. **Given** a user without the role enabling a specific button, **When** the user views the page containing it, **Then** the button is not shown.
4. **Given** a user without the role for an action, **When** the user attempts that action by other means (e.g. a direct link or a direct request), **Then** the action is refused and nothing is changed.
5. **Given** a user in two groups with different profiles, **When** the user signs in, **Then** they have the elements enabled by the roles of both (the sum).
6. **Given** a role reached through two different groups, **When** the user is removed from one of them, **Then** they keep the role through the other.

---

### User Story 2 - Administrator manages users and their groups (Priority: P1)

An administrator adds new users and places them in one or more groups, and can later add or remove them from groups. Roles and profiles cannot be assigned to an individual user; groups are the only way.

**Why this priority**: Without creating users and placing them in groups, no one besides the initial administrator has any access.

**Independent Test**: As administrator, create a user in a group, sign in as that user and verify the access matches the group's profiles.

**Acceptance Scenarios**:

1. **Given** the administrator is on the user management area, **When** they create a user with the required data and choose one or more groups, **Then** the user exists and belongs to those groups.
2. **Given** an existing user, **When** the administrator adds or removes the user from a group, **Then** from the user's next request on, their access reflects the new membership.
3. **Given** the administrator omits a required field or reuses an identifier already used by another user, **When** they try to save, **Then** the user is not created and the problem is explained.
4. **Given** the user management area, **When** the administrator looks for a way to give a role or a profile directly to a user, **Then** there is none.

---

### User Story 3 - Administrator manages groups and their profiles (Priority: P2)

An administrator creates groups, chooses which users belong to them, and associates or removes profiles to and from each group at any time.

**Why this priority**: Groups are the link between people and access; valuable once users exist (Story 2) and access is enforced (Story 1).

**Independent Test**: Create a group, associate a profile, add a user and verify the user gets the roles of that profile; then remove the profile and verify the corresponding elements disappear.

**Acceptance Scenarios**:

1. **Given** the administrator is on the group management area, **When** they create a group with a unique name, **Then** the group exists with no users and no profiles.
2. **Given** an existing group, **When** the administrator associates a profile to it, **Then** all users of the group gain the roles of that profile.
3. **Given** a group with a profile, **When** the administrator removes the profile from it, **Then** all users of the group lose the roles reached only through that profile.
4. **Given** the default groups, **When** the administrator tries to delete or rename them, **Then** it is refused.
5. **Given** a custom group, **When** the administrator deletes it, **Then** its members lose the access reached only through it, and the users themselves are kept.

---

### User Story 4 - Administrator manages profiles and their roles (Priority: P2)

An administrator creates profiles and decides which roles each profile contains, adding or removing roles at any time.

**Why this priority**: Profiles let the organization package roles into reusable sets.

**Independent Test**: Create a profile with two roles, associate it to a group with a user, verify the user gets exactly those two roles' elements; remove one role and verify the element disappears.

**Acceptance Scenarios**:

1. **Given** the profile management area, **When** the administrator creates a profile with a unique name, **Then** the profile exists with no roles.
2. **Given** an existing profile, **When** the administrator adds a role, **Then** all users reaching the profile through their groups gain the element enabled by it.
3. **Given** an existing profile with a role, **When** the administrator removes the role, **Then** all users reaching the profile lose the element, unless they reach the role through another profile.
4. **Given** a profile associated to groups, **When** the administrator tries to delete it, **Then** the system warns which groups are affected and, upon confirmation, removes it from them.

---

### User Story 5 - Administrator manages roles (Priority: P3)

An administrator adds new roles, giving each a name and description and choosing which application element it enables, and can later edit or remove them.

**Why this priority**: The application ships with the roles its features need; extra roles give finer control.

**Independent Test**: Create a role enabling a specific element, add it to a profile associated to a group and verify the group's users gain that element.

**Acceptance Scenarios**:

1. **Given** the role management area, **When** the administrator creates a role with a unique name and selects the element it enables, **Then** the role is available to be added to profiles.
2. **Given** a role used by profiles, **When** the administrator tries to delete it, **Then** the system warns of the impact and, upon confirmation, removes it from those profiles.
3. **Given** a role name already in use, **When** the administrator tries to create another with the same name, **Then** it is refused and explained.

---

### Edge Cases

- A user in no group, or whose groups reach no roles: they can sign in but see no menus or actions, apart from an informative message.
- The last remaining administrator cannot be removed from the "Amministratori" group, deleted, or lose management roles, so the application is never left without an administrator.
- The profile associated to "Amministratori" always holds every role, including roles created later.
- Membership, group, profile or role changes while users are signed in: they apply no later than the user's next request, without signing in again.
- An administrator makes a change that would remove their own access to management areas (for example removing themselves from "Amministratori" when no other path exists): it is refused.
- A role enables an element that is later removed from the application: the role remains but has no effect and is flagged as obsolete to administrators.
- Non-administrators attempt to reach administration areas: access refused, regardless of how they get there.
- Removing a user from a group never deletes the user or their data.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support roles, each enabling exactly one application element (e.g. a menu entry, a button, an action).
- **FR-002**: System MUST support profiles, each being a set of zero or more roles; a role may belong to many profiles.
- **FR-003**: System MUST support groups, each being a set of zero or more users; a user may belong to many groups.
- **FR-004**: System MUST allow associating zero or more profiles to a group; a profile may be associated to many groups.
- **FR-005**: A user's roles MUST be the union of the roles of all profiles associated to all groups the user belongs to.
- **FR-006**: System MUST NOT allow assigning roles or profiles directly to a user.
- **FR-007**: System MUST show a user only the menus, buttons and other controls enabled by the user's roles.
- **FR-008**: System MUST refuse any action not enabled by the user's roles, even when attempted without using the interface, and MUST leave data unchanged.
- **FR-009**: System MUST provide two built-in default groups, "Amministratori" and "Utenti", which cannot be deleted or renamed.
- **FR-010**: The "Amministratori" group MUST be associated to a built-in profile holding all roles, including those created later, giving access to user, group, profile and role management.
- **FR-011**: Every self-registered user MUST be placed in the "Utenti" group (see 003-clerk-auth-sync).
- **FR-012**: Administrators MUST be able to create, view, edit and delete users, and add or remove them to and from groups.
- **FR-013**: Administrators MUST be able to create, view, rename and delete custom groups, and associate or remove profiles to and from them.
- **FR-014**: Administrators MUST be able to create, view, rename and delete custom profiles, and add or remove roles to and from them.
- **FR-015**: Administrators MUST be able to create, view, edit and delete roles, choosing the application element each enables.
- **FR-016**: System MUST enforce unique user identifiers and unique group, profile and role names.
- **FR-017**: System MUST warn about the impact (affected groups or profiles) before deleting a profile or role in use, and require confirmation.
- **FR-018**: System MUST ensure at least one active administrator (a member of "Amministratori") always exists.
- **FR-019**: Changes to users, groups, profiles or roles MUST take effect for affected users no later than their next request, without a new sign-in.
- **FR-020**: Users MUST NOT access user, group, profile or role management unless their roles allow it.
- **FR-021**: System MUST record who made each change to users, groups, profiles and roles and when, and make this history visible to administrators.
- **FR-022**: Administrators MUST be able to block and unblock users (see 003-clerk-auth-sync); the last administrator cannot be blocked.

### Key Entities

- **User**: A person who uses the application. Has an identifier and descriptive data; belongs to zero or more groups. Has no roles of their own.
- **Group**: A named set of users with associated profiles. Built-in groups: "Amministratori" and "Utenti".
- **Profile**: A named set of roles, associated to zero or more groups.
- **Role**: A named permission with description, enabling exactly one application element. Belongs to many profiles.
- **Application element**: A capability of the application (menu, button, action) shown or hidden and allowed or refused according to roles.
- **Change record**: A log entry of who changed a user, group, profile or role, what, and when.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An administrator can create a user and place them in a group in under 1 minute.
- **SC-002**: An administrator can create a profile with roles, associate it to a group and add a user to the group in under 3 minutes.
- **SC-003**: In 100% of tested cases, menus and controls shown to a user match exactly the sum of roles reached through their groups.
- **SC-004**: In 100% of tested cases, an action not allowed by a user's roles is refused, including attempts not made via the interface.
- **SC-005**: A change to membership, group profiles or profile roles is reflected for signed-in users within 1 minute without signing in again.
- **SC-006**: At no time can the application be left without at least one active administrator.

## Assumptions

- Users sign in through the authentication described in 003-clerk-auth-sync; this feature governs what they can do afterwards.
- Administrators can create groups and profiles as well as roles (implied by managing them).
- A role enables one application element; the application defines the available elements and new roles are chosen from them. Creating a role cannot introduce new behavior, only enable what exists.
- Roles are additive: access is the union of roles; there are no "deny" roles.
- Each default group comes with a built-in profile: "Amministratori" with all roles, "Utenti" with only the minimal roles of a regular user. The administrator can change which roles the "Utenti" profile holds.
- An initial administrator exists at first start, in the "Amministratori" group.
- Existing features (e.g. projects and tasks from 001-project-entity) will have their menus and actions gated by roles defined by this feature.
- The initial roles shipped with the application cover the existing features' menus and actions.
