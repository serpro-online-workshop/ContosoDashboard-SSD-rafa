<!--
Sync Impact Report
==================
Version change: (template, unversioned) → 1.0.0
Bump rationale: Initial ratification - all placeholders replaced with concrete principles.

Modified principles (template placeholder → new title):
  - [PRINCIPLE_1_NAME] → I. Training-First, Offline Simplicity
  - [PRINCIPLE_2_NAME] → II. Infrastructure Abstraction for Cloud Migration
  - [PRINCIPLE_3_NAME] → III. Security by Default (Defense in Depth) (NON-NEGOTIABLE)
  - [PRINCIPLE_4_NAME] → IV. Layered Architecture & Consistent Conventions
  - [PRINCIPLE_5_NAME] → V. Verifiable, Warning-Free Changes

Added sections:
  - Technology & Platform Constraints (was [SECTION_2_NAME])
  - Development Workflow & Quality Gates (was [SECTION_3_NAME])
  - Governance (filled from [GOVERNANCE_RULES])

Removed sections: none

Templates / guidance files:
  - This file mirrors /.specify/memory/constitution.md (repo root Spec Kit install); keep both identical.
  - ContosoDashboard/.specify/templates/plan-template.md ✅ compatible (Constitution Check gate reads this file)
  - ContosoDashboard/.specify/templates/spec-template.md ✅ compatible
  - ContosoDashboard/.specify/templates/tasks-template.md ✅ compatible (tests optional unless requested)
  - README.md (repo root) ✅ updated (.NET 10 references)
  - Paths in this document (README.md, StakeholderDocs/, specs/) are relative to the repository root.

Deferred TODOs: none
-->

# ContosoDashboard Constitution

## Core Principles

### I. Training-First, Offline Simplicity

ContosoDashboard exists to teach Spec-Driven Development. Every feature MUST run fully offline on a
developer machine or Codespace with nothing more than the .NET SDK installed.

- Features MUST NOT require cloud subscriptions, external services, network APIs, or separately
  installed servers (databases, message brokers, identity providers) to build, run, or demo.
- Persistent data MUST live in the local SQLite database; uploaded or generated files MUST live on
  the local filesystem.
- Prefer the simplest design that satisfies the spec (YAGNI). New projects, layers, frameworks, or
  NuGet dependencies MUST be justified in the plan's Complexity Tracking table.
- The application remains a training sample: known limitations MAY be accepted when they are
  documented (README "Known Limitations") rather than silently ignored.

**Rationale**: Students must be able to complete the workshop anywhere, with no cost and no setup
friction; complexity obscures the SDD lessons being taught.

### II. Infrastructure Abstraction for Cloud Migration

Infrastructure dependencies MUST be reached through interfaces registered with dependency injection
so that local implementations can be swapped for Azure services without touching business logic.

- File storage MUST go through an `IFileStorageService`-style abstraction (local implementation now,
  Azure Blob Storage later). Pages and business services MUST NOT call `System.IO` directly for
  user content.
- Data access MUST go through EF Core `ApplicationDbContext` and service classes; provider-specific
  SQL, raw SQL, or SQLite-only features MUST NOT leak into business logic, so that moving from
  SQLite to Azure SQL only changes the provider registration and connection string.
- Stored paths and keys MUST be relative and portable (e.g., `{userId}/{projectId|personal}/{guid}.{ext}`)
  so they remain valid as blob names after migration.
- Configuration (connection strings, storage roots, size limits) MUST come from `appsettings.json`,
  not hard-coded values.

**Rationale**: The workshop demonstrates an industry-standard migration path; abstractions make the
production swap a configuration/implementation change instead of a rewrite.

### III. Security by Default (Defense in Depth) (NON-NEGOTIABLE)

Every feature MUST preserve and extend the existing layered security model.

- Every page or endpoint exposing user data MUST carry `[Authorize]` (with a role policy where
  appropriate: Employee, TeamLead, ProjectManager, Administrator).
- Every service method that reads or mutates a resource by ID MUST verify the requesting user's
  right to that resource (ownership, project membership, department, or role) to prevent IDOR.
  UI-only checks are never sufficient.
- Authorization decisions MUST use claims (NameIdentifier, Name, Email, Role, Department); any new
  claim a feature depends on MUST be added to the login flow.
- User-supplied input MUST be validated server-side. For files: enforce size limits, validate
  extensions against an allow-list, never use user-supplied names in filesystem paths (use GUIDs),
  store outside `wwwroot`, and serve only through authorized endpoints.
- Existing security headers (CSP, X-Frame-Options, etc.) MUST NOT be weakened; any CSP relaxation
  MUST be justified in the plan.
- Dependencies MUST NOT have known vulnerabilities (`dotnet list package --vulnerable
  --include-transitive` reports none).

**Rationale**: Security is a core teaching objective of this sample; a single missing service-level
check reintroduces the IDOR class of bugs the application is designed to demonstrate preventing.

### IV. Layered Architecture & Consistent Conventions

New code MUST follow the existing project structure and patterns.

- Layers: `Models/` (entities), `Data/` (`ApplicationDbContext`, relationships, indexes, seed data),
  `Services/` (`IXxxService` + `XxxService`, registered as scoped), `Pages/` and `Shared/` (Blazor
  components and Razor Pages). Pages MUST NOT query `ApplicationDbContext` directly.
- All I/O MUST be async (`async`/`await`, `...Async` EF Core methods).
- Related data MUST be loaded deliberately (`.Include()`) to avoid N+1 queries; frequently filtered
  columns MUST be indexed in `OnModelCreating`.
- Entity primary keys MUST be `int` identity keys, consistent with existing User/Project/Task keys;
  enumerations persisted as text MUST be called out in the data model.
- Multi-step operations that touch storage and the database MUST be ordered to avoid orphaned data
  (e.g., generate unique path → save file → save metadata; compensate on failure).
- UI MUST use Bootstrap 5.3 and Bootstrap Icons consistent with existing pages.

**Rationale**: Consistency lets students compare generated code against the existing baseline and
keeps the codebase navigable as features are added by different spec runs.

### V. Verifiable, Warning-Free Changes

Every change MUST be demonstrably working before it is considered done.

- `dotnet build` MUST succeed with 0 errors and MUST NOT introduce new compiler or NuGet warnings
  (pre-existing warnings are tracked, not multiplied).
- The application MUST start from a fresh database (delete `ContosoDashboard.db*`, run
  `dotnet run`) with `EnsureCreated()` seeding successfully and no startup exceptions in the log.
- Each user story MUST be independently verifiable using the seeded mock users, including at least
  one negative authorization check (a user who must NOT see or change the resource).
- Automated tests (xUnit) are OPTIONAL and MUST be written when the feature specification requests
  them; when written, they MUST fail before the implementation and pass after.

**Rationale**: The repository has no automated test suite by design; explicit build, startup, and
role-based verification gates keep quality measurable without adding training overhead.

## Technology & Platform Constraints

- **Runtime**: .NET 10 (`net10.0`), ASP.NET Core Blazor Server with Razor Pages for login/logout.
- **Data**: Entity Framework Core 10 with the SQLite provider; connection string
  `Data Source=ContosoDashboard.db`; schema created via `EnsureCreated()` for development. Schema
  changes MUST keep the seed data valid. SQLite database files MUST NOT be committed.
- **Authentication**: Cookie-based mock authentication with claims; Microsoft Entra ID is the
  documented production path and MUST NOT be required to run the app.
- **File storage**: Local filesystem under a configurable root outside `wwwroot` (e.g.,
  `AppData/uploads`), accessed only via the storage abstraction.
- **UI**: Bootstrap 5.3 and Bootstrap Icons; no additional client-side frameworks or build tooling
  (npm, bundlers) without a constitution amendment.
- **Packages**: Microsoft-supported NuGet packages on the latest patch for the target framework,
  with no known vulnerabilities.
- **Performance baseline**: Typical pages SHOULD render in under 2 seconds and typical uploads
  (under 10 MB) SHOULD complete in under 30 seconds on a developer machine.

## Development Workflow & Quality Gates

1. **Specify** (`/speckit.specify`): Specs describe user value and MUST stay technology-agnostic;
   stakeholder documents in `StakeholderDocs/` are the source of requirements.
2. **Clarify** (`/speckit.clarify`): Resolve `[NEEDS CLARIFICATION]` markers before planning.
3. **Plan** (`/speckit.plan`): The plan's **Constitution Check** MUST evaluate each of Principles
   I-V and the Technology & Platform Constraints; any violation MUST be recorded in Complexity
   Tracking with justification, or the design changed.
4. **Tasks** (`/speckit.tasks`): Tasks are grouped by user story; security tasks (authorization
   attributes, service-level checks, input validation) MUST appear explicitly, not implicitly.
5. **Implement** (`/speckit.implement`): Each completed story MUST pass the Principle V gates.
6. **Documentation**: README sections (features, project structure, configuration, known
   limitations) MUST be updated in the same change that alters them.
7. **Commits**: Use Conventional Commit prefixes (`feat:`, `fix:`, `docs:`, `chore:`).

## Governance

- This constitution supersedes other practices and guidance in this repository. Where the README,
  stakeholder documents, or generated artifacts conflict with it, the constitution wins until
  amended.
- Amendments MUST be made through `/speckit.constitution`, MUST include an updated Sync Impact
  Report, and MUST propagate changes to dependent templates in `.specify/templates/` and to
  `README.md` where relevant.
- Versioning follows semantic versioning: MAJOR for removed or redefined principles, MINOR for new
  principles/sections or materially expanded guidance, PATCH for clarifications and wording.
- Compliance review: every plan's Constitution Check and every pull request review MUST verify
  adherence to Principles I-V; unjustified complexity or skipped security checks block merge.
- Runtime development guidance lives in `README.md`; feature requirements live in
  `StakeholderDocs/` and `specs/`.

**Version**: 1.0.0 | **Ratified**: 2026-10-20 | **Last Amended**: 2026-10-20
