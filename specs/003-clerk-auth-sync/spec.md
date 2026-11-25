# Feature Specification: Clerk Authentication and User Synchronization

**Feature Branch**: `003-clerk-auth-sync`

**Created**: 2026-11-25

**Status**: Draft

**Input**: User description: "il login e l'autenticazione avviene attraverso Clerk. quindi quando creo un utente sull'applicativo devo crearlo automaticamente su clerck e poi eventualmente se lo cancello"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Sign in through Clerk (Priority: P1)

Users sign in to the application using Clerk, the approved authentication provider (see constitution v2.0.0, "Approved Exceptions"). After a successful sign-in, the application recognizes the person as one of its own users and applies the access of that user's profile (see 002-roles-profiles-access). Sign-out ends the session.

**Why this priority**: Without authentication nobody can use the application; it is the entry point for all other features.

**Independent Test**: With a user that exists both in the application and in Clerk, sign in, verify the application opens with the access of the user's profile, then sign out and verify protected pages are no longer reachable.

**Acceptance Scenarios**:

1. **Given** an application user who also exists in Clerk, **When** they sign in with valid credentials, **Then** they reach the application with the menus and actions of their profile.
2. **Given** an unauthenticated visitor, **When** they open any application page, **Then** they are asked to sign in and no application data is shown.
3. **Given** a person who authenticates successfully with Clerk but has no corresponding application user, **When** they try to enter, **Then** access is refused with a message that they are not authorized.
4. **Given** a signed-in user, **When** they sign out, **Then** the session ends and protected pages require signing in again.

---

### User Story 2 - Creating a user also creates it in Clerk (Priority: P1)

When an administrator creates a user in the application, the user is automatically created in Clerk too, so the new user can sign in without any manual step on Clerk. The administrator never needs to open Clerk.

**Why this priority**: It makes user administration a single operation and guarantees that every application user can actually sign in.

**Independent Test**: As administrator, create a user; verify the user exists in the application and in Clerk, and that the new user can sign in.

**Acceptance Scenarios**:

1. **Given** the administrator submits valid data for a new user, **When** the user is saved, **Then** the user exists in the application and in Clerk and can sign in.
2. **Given** Clerk is unavailable or rejects the creation, **When** the administrator saves a new user, **Then** no user is created in the application, the administrator is told the reason, and no partial user is left in either system.
3. **Given** the email is already used by a user in Clerk or in the application, **When** the administrator tries to create the user, **Then** the creation is refused and explained.

---

### User Story 3 - Deleting a user also deletes it in Clerk (Priority: P2)

When an administrator deletes a user in the application, the corresponding Clerk account is deleted too, so the person can no longer sign in and active sessions end.

**Why this priority**: Prevents former users from keeping access; important for security, but it follows creation in the lifecycle.

**Independent Test**: Delete a user as administrator; verify the user is gone from the application and from Clerk, and that sign-in attempts fail.

**Acceptance Scenarios**:

1. **Given** an existing user, **When** the administrator deletes them, **Then** the user is removed from the application and from Clerk, and they can no longer sign in.
2. **Given** the deleted user is signed in at the time, **When** their next request arrives, **Then** it is refused and they are asked to sign in (which will fail).
3. **Given** Clerk is unavailable or rejects the deletion, **When** the administrator deletes a user, **Then** the user is not deleted in the application either, and the administrator is told to retry.

---

### User Story 4 - Keep users aligned and recover from mismatches (Priority: P3)

Administrators can see whether each application user is correctly linked to a Clerk account, and can fix mismatches (for example a user existing only on one side) without creating duplicates.

**Why this priority**: Failures and manual interventions on Clerk can leave the two systems misaligned; this makes the situation visible and recoverable.

**Independent Test**: Remove a user directly in Clerk, then verify the administrator sees that user flagged as not linked and can re-create the Clerk account.

**Acceptance Scenarios**:

1. **Given** an application user without a Clerk account, **When** the administrator views users, **Then** the user is flagged as "not linked".
2. **Given** a flagged user, **When** the administrator chooses to re-create the Clerk account, **Then** the user becomes linked and can sign in.

---

### Edge Cases

- Clerk becomes unavailable during creation or deletion: the operation fails as a whole, with no half-created or half-deleted user (see FR-005).
- An administrator tries to delete their own user, or the last administrator: refused (rule from 002-roles-profiles-access, FR-014).
- A user is created or deleted directly in Clerk (outside the application): the application does not grant access to Clerk-only people and flags application-only users as not linked.
- The same email in different letter case: treated as the same user.
- A user's email is changed in the application: the change is applied to Clerk too, or refused as a whole.
- Two administrators create the same user at the same time: only one succeeds, the other is told the user already exists.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Sign-in and authentication of application users MUST be performed through Clerk.
- **FR-002**: System MUST allow access only to people authenticated by Clerk who correspond to an existing application user; all others MUST be refused.
- **FR-003**: When an administrator creates a user, System MUST automatically create the corresponding account in Clerk, with no manual step.
- **FR-004**: When an administrator deletes a user, System MUST automatically delete the corresponding account in Clerk and end that user's access.
- **FR-005**: Creation and deletion MUST be all-or-nothing: if the operation fails in either the application or Clerk, the state in both systems MUST remain as before, and the administrator MUST be told the reason.
- **FR-006**: System MUST link each application user to exactly one Clerk account, identified by the user's email, and refuse creation when the email is already in use.
- **FR-007**: Application users MUST NOT be able to self-register; users only exist when created by an administrator.
- **FR-008**: When a user's email is changed by an administrator, System MUST apply the change in Clerk as well, or refuse the whole change.
- **FR-009**: System MUST show administrators which users are not linked to a Clerk account and allow re-creating the link.
- **FR-010**: Profiles, roles and access rules MUST be stored and decided by the application, not by Clerk.
- **FR-011**: System MUST record creation, deletion and synchronization failures for users in the change history (see 002-roles-profiles-access, FR-017).
- **FR-012**: Deleting a user MUST take effect on access no later than the user's next request.
- **FR-013**: The application MUST keep Clerk-specific behavior isolated so the identity provider can be replaced in the future without changing roles, profiles or application data (constitution v2.0.0).

### Key Entities

- **User**: An application user (see 002-roles-profiles-access) with an email and a link to exactly one Clerk account.
- **Clerk account**: The sign-in identity held by Clerk for a user; created and deleted together with the user.
- **Link status**: Whether a user is correctly linked to a Clerk account (linked / not linked).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An administrator creates a user who can sign in immediately, in a single operation of under 1 minute and with no action outside the application.
- **SC-002**: A deleted user cannot sign in, and their active session is refused, within 1 minute of deletion.
- **SC-003**: In 100% of tested failure cases (Clerk unavailable, duplicate email), no user is left existing in only one of the two systems after creation or deletion.
- **SC-004**: 100% of people authenticated by Clerk without a matching application user are refused access.
- **SC-005**: An administrator can identify all non-linked users from a single view and repair one in under 1 minute.

## Assumptions

- Clerk is the single approved identity provider; its use as a hard requirement for sign-in is permitted by constitution v2.0.0, limited to authentication and user management.
- The sign-in method offered to users (e.g. email and password, email codes) is configured in Clerk and is not decided by this feature.
- The application user's email is the key linking the two systems; email is unique across users.
- The first administrator is created at initial setup (outside this feature) and linked to Clerk.
- Roles, profiles and permissions are those of 002-roles-profiles-access; Clerk stores only identity and credentials.
- Deleting a user is permanent in both systems; no recovery period is provided.
- Clerk availability is outside the project's control; a Clerk outage prevents sign-in, user creation and user deletion, but is reported clearly.
