<!--
Sync Impact Report
==================
Version change: (unversioned template) → 1.0.0
Bump rationale: initial ratification; all template placeholders replaced with project content.

Modified principles (template slot → new title):
  - [PRINCIPLE_1_NAME] → I. Spec-Driven Traceability
  - [PRINCIPLE_2_NAME] → II. Test-First (NON-NEGOTIABLE)
  - [PRINCIPLE_3_NAME] → III. Simplicity / YAGNI
  - [PRINCIPLE_4_NAME] → IV. Security & Observability
  - [PRINCIPLE_5_NAME] → V. Explicit Service Boundaries

Added sections:
  - Technology Stack & Architecture Constraints (template SECTION_2)
  - Development Workflow & Quality Gates (template SECTION_3)
  - Governance (filled)

Removed sections: none

Deferred items / follow-up TODOs:
  - Project name "Online Chess Platform" was derived from the user's description; confirm or rename.
  - Concrete Java, Spring Boot, Node, and React versions are intentionally not pinned here; they
    live in the build files (see Technology Stack section).

Templates: not modified by this command (they read the constitution at runtime).
-->

# Online Chess Platform Constitution

## Core Principles

### I. Spec-Driven Traceability

Every change MUST trace back to a requirement in a feature specification (`spec.md`) and a task
in that feature's `tasks.md`.

- No production code is written without a spec and a task that calls for it.
- Commits and pull requests MUST reference the feature and task IDs they implement.
- When implementation reveals a gap or a change in behavior, the spec MUST be updated first,
  then the plan and tasks, then the code.

**Rationale**: The specs stay the single source of truth for behavior across backend and
frontend, so reviewers can check what was built against what was asked for.

### II. Test-First (NON-NEGOTIABLE)

Tests MUST be written and observed failing before the implementation that makes them pass.

- Red-Green-Refactor is strictly enforced for both backend and frontend.
- The backend MUST have unit tests (JUnit 5) for domain and service logic. Chess rules (legal
  moves, check, checkmate, stalemate, draw conditions) MUST be covered exhaustively.
- The frontend MUST have unit tests for components, hooks, and client-side state logic.
- Every bug fix MUST start with a test that reproduces the bug.
- Code that is merged without tests for its behavior is a constitution violation.

**Rationale**: Chess logic and multiplayer game state have many edge cases. Writing tests first
pins down correct behavior before the code exists and prevents regressions.

### III. Simplicity / YAGNI

Build only what the current spec requires.

- No speculative abstractions, extension points, or configuration for features that have not
  been specified.
- Prefer the standard Spring Boot and React mechanisms over custom frameworks or extra
  libraries.
- Any added complexity (a new service, a new dependency, a new architectural pattern) MUST be
  justified in the Complexity Tracking section of `plan.md`, including the simpler alternative
  that was rejected and why.

**Rationale**: Every extra layer costs effort to maintain and test. Simplicity keeps the
monorepo approachable and the specs honest.

### IV. Security & Observability

Account and game data MUST be protected, and system behavior MUST be diagnosable.

- Passwords MUST be stored only as hashes from an adaptive algorithm (e.g. BCrypt or Argon2).
  Plaintext or reversible storage is forbidden.
- Every non-public endpoint and real-time channel MUST require authentication. Users MUST only be
  able to act on their own account and on games they participate in.
- The server is authoritative: every move MUST be validated on the backend. Client-side
  validation exists only for user experience and is never trusted.
- All external input MUST be validated at the API boundary.
- Secrets (database credentials, signing keys) MUST NOT be committed. They are supplied through
  environment variables or Docker secrets.
- The backend MUST use structured logging that includes a request or correlation ID. Logs MUST
  NOT contain passwords, tokens, or other secrets.
- Errors returned to clients MUST be actionable and MUST NOT leak stack traces or internal
  details.

**Rationale**: The platform handles user credentials and competitive play. Trusting the client
would enable cheating, and missing logs make production problems impossible to diagnose.

### V. Explicit Service Boundaries

The backend and frontend are independently buildable and testable units that communicate only
through a documented API contract.

- The API contract (REST and, where needed, WebSocket messages) MUST be defined in the feature's
  `contracts/` before either side implements against it.
- The frontend MUST NOT depend on backend internals, and the backend MUST NOT depend on frontend
  code. Shared knowledge lives only in the contract.
- Breaking contract changes MUST be called out in the spec and plan, and both sides MUST be
  updated in the same feature.
- Each unit MUST build, run its own tests, and start in its own Docker container.

**Rationale**: Clear boundaries let either side change internally without breaking the other.
They also keep the Docker-composed system predictable.

## Technology Stack & Architecture Constraints

- **Repository layout**: A single monorepo containing a `backend/` project and a `frontend/`
  project, each with its own build and test configuration.
- **Backend**: Java with Spring Boot. Unit tests use JUnit 5, with Mockito where test doubles are
  needed.
- **Frontend**: React with TypeScript. TypeScript `strict` mode MUST be enabled, and `any` is
  disallowed unless justified in code review.
- **Integration**: The backend and frontend each have their own Dockerfile and are wired
  together with Docker Compose. The full system MUST start locally with a single
  `docker compose up`.
- **Versions**: Language, framework, and tool versions are pinned in the build files (Maven or
  Gradle, `package.json` and its lockfile, Dockerfiles), not in this constitution. Version
  upgrades are planned features and follow the normal spec workflow.
- **New technology**: Adding a new language, framework, datastore, or infrastructure component
  requires justification under Principle III in the relevant `plan.md`.

## Development Workflow & Quality Gates

- **Workflow**: Features follow the Spec Kit flow: `/speckit-specify` → `/speckit-clarify` (when
  the spec is ambiguous) → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`.
- **Constitution Check**: Each `plan.md` MUST pass the Constitution Check gate before design
  starts and again after design. Violations MUST be resolved or recorded with justification in
  Complexity Tracking.
- **Merge gates**: A change may merge only when all of the following hold:
  1. All backend and frontend unit tests pass.
  2. Both Docker images build successfully.
  3. Linting and type checks pass, including the TypeScript compiler with no errors.
  4. At least one reviewer has approved and confirmed compliance with this constitution.
- **Pull requests**: Each pull request is scoped to one feature or a coherent subset of its
  tasks, and MUST reference the spec and task IDs it implements.

## Governance

This constitution supersedes all other development practices for this repository. When another
guideline conflicts with it, the constitution wins.

- **Amendments**: Amendments are proposed through `/speckit-constitution`, documented in the Sync
  Impact Report, reviewed in a pull request, and approved by the project maintainers before
  merge. Amendments that change obligations for existing work MUST include a migration note.
- **Versioning**: This document follows semantic versioning:
  - MAJOR: principles or governance rules are removed or redefined in a backward-incompatible
    way.
  - MINOR: a principle or section is added, or guidance is materially expanded.
  - PATCH: clarifications, wording, and typo fixes.
- **Compliance review**: Every pull request review MUST verify compliance with these principles.
  Unjustified complexity or skipped tests are grounds to block a merge. Compliance with the
  constitution is also re-checked at each `/speckit-plan` and `/speckit-analyze` run.

**Version**: 1.0.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-06
