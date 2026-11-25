# Feature Specification: Roles, Profiles and Access Control

**Feature Branch**: `002-roles-profiles-access`

**Created**: 2026-11-25

**Status**: Draft

**Input**: User description: "nell'applicativo ci devono essere ruoli e profili. Un profilo e' un insieme di ruoli. Ogni utente appartiene ad un profilo e in base ai ruoli che ha quel profilo puo' vedere un menu fare azioni ecc. Ogni ruolo abilita una cosa che puo essere un menu o un bottoe ecc. il profilo amministratore ha la possibilita di inserire ruoli associare ai profili i ruoli aggiungere i ruoli e aggiungere gli utenti"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access adapts to the user's profile (Priority: P1)

Every user belongs to exactly one profile. A profile is a set of roles, and each role enables one specific element of the application (a menu entry, a button, an action, and so on). When a user works in the application, they see only the menus and controls enabled by the roles of their profile, and can perform only the corresponding actions.

**Why this priority**: This is the core value of the feature: without enforcement of roles on what users see and do, profiles and roles have no effect.

**Independent Test**: Create two users with two profiles holding different roles; sign in as each and verify that menus, buttons and actions differ exactly according to the roles of each profile.

**Acceptance Scenarios**:

1. **Given** a user whose profile includes the role enabling the "Projects" menu, **When** the user opens the application, **Then** the "Projects" menu entry is visible and usable.
2. **Given** a user whose profile does not include the role enabling the "Projects" menu, **When** the user opens the application, **Then** that menu entry is not shown.
3. **Given** a user whose profile lacks the role enabling a specific button, **When** the user views the page containing it, **Then** the button is not shown.
4. **Given** a user whose profile lacks the role for an action, **When** the user attempts that action by other means (e.g. opening a direct link or sending the request directly), **Then** the action is refused and nothing is changed.
5. **Given** a profile that includes several roles, **When** a user of that profile signs in, **Then** all the elements enabled by any of those roles are available.

---

### User Story 2 - Administrator manages users (Priority: P1)

An administrator adds new users and assigns each of them to a profile. The administrator can also change a user's profile later.

**Why this priority**: Without the ability to create users and assign them to profiles, no one besides the initial administrator can use the application.

**Independent Test**: As administrator, create a user assigned to a profile, then sign in as that user and verify the access matches the profile.

**Acceptance Scenarios**:

1. **Given** the administrator is on the user management area, **When** they create a user with the required data and choose a profile, **Then** the user exists and belongs to that profile.
2. **Given** an existing user, **When** the administrator changes the user's profile, **Then** from the user's next request on, their access reflects the new profile.
3. **Given** the administrator omits a required field or reuses an identifier already used by another user, **When** they try to save, **Then** the user is not created and the problem is explained.

---

### User Story 3 - Administrator manages profiles and their roles (Priority: P2)

An administrator creates profiles and decides which roles each profile contains, adding or removing roles from a profile at any time.

**Why this priority**: It allows the organization to shape access to its own needs; it is valuable once users can exist (Story 2) and access is enforced (Story 1).

**Independent Test**: Create a profile, associate two roles to it, assign a user to it and verify the user gets exactly those two roles' elements; then remove one role and verify the corresponding element disappears.

**Acceptance Scenarios**:

1. **Given** the administrator is on the profile management area, **When** they create a profile with a unique name, **Then** the profile exists with no roles.
2. **Given** an existing profile, **When** the administrator associates a role to it, **Then** all users of that profile gain the element enabled by that role.
3. **Given** an existing profile with a role, **When** the administrator removes the role from it, **Then** all users of that profile lose the element enabled by that role.
4. **Given** a profile with assigned users, **When** the administrator tries to delete it, **Then** the deletion is refused and the administrator is told to reassign the users first.

---

### User Story 4 - Administrator manages roles (Priority: P3)

An administrator adds new roles, giving each a name and description and choosing which application element it enables (a menu entry, a button, an action, and so on), and can later edit or remove them.

**Why this priority**: The application ships with the roles needed for its features; creating additional roles is a refinement that provides finer-grained control.

**Independent Test**: Create a role enabling a specific element, add it to a profile and verify users of that profile gain that element.

**Acceptance Scenarios**:

1. **Given** the administrator is on the role management area, **When** they create a role with a unique name and select the element it enables, **Then** the role is available to be associated to profiles.
2. **Given** a role used by one or more profiles, **When** the administrator tries to delete it, **Then** the system warns of the impact and, upon confirmation, removes the role from those profiles.
3. **Given** a role name already in use, **When** the administrator tries to create another with the same name, **Then** it is refused and explained.

---

### Edge Cases

- A user without a profile, or whose profile has no roles: they can sign in but see no menus or actions, apart from an informative message.
- The last remaining administrator cannot be moved to another profile, removed, or have the administrator profile's management roles removed, so the application is never left without an administrator.
- The administrator profile always holds every role, including roles created later.
- A profile's roles are changed while its users are signed in: the change applies no later than the user's next request/page load, without needing to sign in again.
- An administrator tries to edit their own profile in a way that removes their access to user/profile/role management: it is refused.
- A role enables an element that is later removed from the application: the role remains but has no effect and is flagged as obsolete to administrators.
- Non-administrators attempt to reach administration areas: access refused, regardless of how they get there.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support roles, each enabling exactly one application element (e.g. a menu entry, a button, an action).
- **FR-002**: System MUST support profiles, each being a set of zero or more roles; a role may belong to many profiles.
- **FR-003**: Every user MUST belong to exactly one profile.
- **FR-004**: System MUST show a user only the menus, buttons and other controls enabled by the roles of the user's profile.
- **FR-005**: System MUST refuse any action not enabled by the roles of the user's profile, even when attempted without using the interface, and MUST leave data unchanged.
- **FR-006**: System MUST provide an administrator profile that holds all roles, including those created later, and can access user, profile and role management.
- **FR-007**: Administrators MUST be able to create, view, edit and delete users, and assign or change each user's profile.
- **FR-008**: Administrators MUST be able to create, view, rename and delete profiles.
- **FR-009**: Administrators MUST be able to associate roles to a profile and remove roles from it.
- **FR-010**: Administrators MUST be able to create, view, edit and delete roles, choosing the application element each enables.
- **FR-011**: System MUST enforce unique user identifiers, unique profile names and unique role names.
- **FR-012**: System MUST refuse deletion of a profile that still has users, telling the administrator to reassign them first.
- **FR-013**: System MUST warn about the impact (affected profiles) before deleting a role in use, and require confirmation.
- **FR-014**: System MUST ensure at least one administrator user always exists.
- **FR-015**: Changes to users, profiles or roles MUST take effect for affected users no later than their next request, without requiring a new sign-in.
- **FR-016**: Users MUST NOT be able to access user, profile or role management unless their profile holds the corresponding roles.
- **FR-017**: System MUST record who made each change to users, profiles and roles and when, and make this history visible to administrators.

### Key Entities

- **User**: A person who uses the application. Has an identifier and descriptive data; belongs to exactly one profile.
- **Profile**: A named set of roles. Has many users and many roles. One built-in profile is the administrator profile.
- **Role**: A named permission with description, enabling exactly one application element (menu entry, button, action, ...). Can belong to many profiles.
- **Application element**: A capability of the application (a menu, a button, an action) that can be shown or hidden and allowed or refused according to roles.
- **Change record**: A log entry of who changed a user, profile or role, what changed, and when.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An administrator can create a user and assign them a profile in under 1 minute.
- **SC-002**: An administrator can create a profile, associate roles to it and assign a user to it in under 3 minutes.
- **SC-003**: In 100% of tested cases, menus and controls shown to a user match exactly the roles of their profile.
- **SC-004**: In 100% of tested cases, an action not allowed by a user's roles is refused, including attempts not made via the interface.
- **SC-005**: A change to a profile's roles is reflected for its signed-in users within 1 minute without them signing in again.
- **SC-006**: At no time can the application be left without at least one active administrator.

## Assumptions

- Users already have a way to sign in (identity verification is handled by authentication); this feature governs what they can do after signing in.
- Administrators can create profiles as well as assigning roles to them (implied by "associate roles to profiles").
- A role enables one application element; the application defines the set of available elements, and new roles are chosen from it. Creating a role cannot introduce new application behavior, only enable what exists.
- A user belongs to exactly one profile at a time; the same person needing a different mix of access uses a different profile.
- The administrator profile is built-in, cannot be deleted or have roles removed, and an initial administrator exists at first start.
- Roles are additive: a profile's access is the union of its roles; there are no "deny" roles.
- Existing features (e.g. projects and tasks from 001-project-entity) will have their menus and actions gated by roles defined by this feature.
- The initial roles shipped with the application cover the existing features' menus and actions.
