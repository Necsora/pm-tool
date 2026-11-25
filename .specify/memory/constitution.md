# pm-tool Constitution

## Core Principles

### I. Open Source Software Only (NON-NEGOTIABLE)
All software that is part of pm-tool MUST be open source. This covers the project's own source
code and every runtime, build-time, and development dependency (libraries, frameworks, tools,
services that are self-hosted as part of the product).

- The project's own code MUST be published under the GNU Affero General Public License v3.0
  (AGPL-3.0).
- Every dependency MUST be distributed under an OSI-approved open source license that is
  compatible with AGPL-3.0.
- Proprietary, source-available, or closed-source components MUST NOT be added to the product.
- The product MUST remain buildable and runnable using open source components alone.

Rationale: openness guarantees that the code can be inspected, modified, self-hosted, and
maintained without dependence on a single vendor.

## Licensing and Dependency Constraints

- Each new dependency MUST have its license identified (SPDX identifier preferred) before it is
  adopted, and that license MUST be recorded in the feature's plan or in a dependency manifest.
- A dependency whose license cannot be determined or is not OSI-approved MUST be rejected or
  replaced.
- Third-party hosted proprietary services (e.g., SaaS APIs) MUST NOT be a hard requirement for
  the product to function; if integrated, they MUST be optional and replaceable with an open
  source alternative, except for the approved exception below.
- The repository MUST contain the full AGPL-3.0 license text in a `LICENSE` file at its root.
- Dependencies under licenses incompatible with AGPL-3.0 (e.g., GPL-2.0-only, SSPL, BUSL) MUST
  NOT be added.
- Because AGPL-3.0 applies to network use, the corresponding source of any deployed version MUST
  be offered to its users.

## Approved Exceptions

### Clerk (authentication and user identity)
Clerk (clerk.com), a proprietary hosted SaaS, is approved as the authentication and user-identity
provider of pm-tool, and MAY be a hard requirement for sign-in and user lifecycle management.

- The exception is limited to Clerk and to authentication and user management (sign-in, identity,
  and creating and deleting the corresponding user accounts). It does not extend to any other
  feature, data, or service.
- All other proprietary or hosted services remain prohibited as hard requirements, as stated in
  the constraints above.
- Authorization (roles, profiles, access control) and all application data MUST remain in
  pm-tool's own open source code and storage; Clerk MUST NOT be the sole holder of them.
- Clerk-specific code SHOULD be isolated behind a narrow boundary so that the provider can later
  be replaced.

Rationale: delegating authentication to a specialized managed provider reduces security risk and
effort, and the narrow scope limits dependence on a single vendor.

## Governance

This constitution supersedes other project practices. Amendments require a written change to this
file, a version bump, and an updated Last Amended date. Versioning follows semantic versioning:
MAJOR for removals or redefinitions of principles, MINOR for new principles or materially expanded
guidance, PATCH for clarifications and wording. Every specification, plan, and review MUST verify
compliance with the principles above; any deviation MUST be explicitly justified and approved
through an amendment before being merged.

**Version**: 2.0.0 | **Ratified**: 2026-11-25 | **Last Amended**: 2026-11-25
