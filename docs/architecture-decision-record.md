# Architecture decision record

OpenCode Observatory — Phase 1 architecture decisions

- **Status date:** 2026-10-31
- **Decision scope:** the architecture and engineering direction for Phases 2–7
- **Implementation state:** these decisions are planning decisions; Phase 2 has
  not started
- **Related evidence:** `docs/phase-1-audit.md`
- **Related requirements:** `docs/requirements-traceability.md`

## How to read this record

Each decision records the context, the decision, the rationale, the
consequences and the evidence that will prove it. A decision can be reopened
when its stated trigger occurs. Reopening an ADR records the new evidence and
supersession explicitly; it never silently changes an earlier requirement.

Status vocabulary:

- **Accepted** — the decision is the current architecture direction.
- **Proposed** — the direction is selected in principle, but a bounded spike
  must provide the named evidence before the implementation commits to it.
- **Rejected** — the option is not part of the initial architecture. A rejected
  option may be reconsidered only with new evidence and a superseding ADR.

## Decision index

| ADR | Topic | Status | Primary consequence |
|---|---|---|---|
| ADR-001 | Build strategy | Accepted | Clean-room TypeScript application, not a candidate fork |
| ADR-002 | Candidate code reuse | Accepted (candidate reuse Rejected) | No candidate code or assets are copied |
| ADR-003 | Source access | Accepted | Read-only SQLite primary, optional GET-only API supplement |
| ADR-004 | Dashboard database | Accepted | Separate SQLite store with its own migrations |
| ADR-005 | Canonical identity and deduplication | Accepted | V2 precedence, legacy-only retention, request-level ownership |
| ADR-006 | Cost ledger model | Accepted | Six separate provenance-aware ledgers |
| ADR-007 | Optional AI boundary | Accepted | AI off by default, opt-in, redacted, typed and evidence-linked |
| ADR-008 | Future hosting boundary | Accepted | Local-first now; optional explicit sync later |
| ADR-009 | Security posture | Accepted | Loopback, Host and Origin checks, no default upload or telemetry |
| ADR-010 | Application stack | Accepted | Next.js, strict TypeScript, Tailwind v4, shadcn/ui, Lucide, Recharts |
| ADR-011 | Package manager | Accepted | npm workspaces |
| ADR-012 | SQLite client | Proposed | `node:sqlite` must pass a Windows spike before commitment |
| ADR-013 | Fixture strategy | Accepted | Synthetic legacy, V2, migration-overlap and relationship-edge fixtures are mandatory |
| ADR-014 | Project and third-party licensing | Accepted | MIT project; explicit attribution when external material is introduced |
| ADR-015 | Living documentation and phase gates | Accepted | Versioned docs and traceability accompany each approved phase |

---

## ADR-001 — Use a clean-room TypeScript build

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-ARCH-001` through `PRD-ARCH-014`,
`PRD-ARCH-010` through `PRD-ARCH-013`,
`PRD-RULE-002`, `PRD-RULE-008`, `PRD-PHASE-001` through `PRD-PHASE-009`

### Context

The repository is a workflow starter kit with no application. The Master
Prompt prefers Next.js and TypeScript unless the audit gives a strong reason
to choose otherwise. The host has Node 24.21.0 and npm 11.19.0, but no Go
toolchain, and the global pnpm and Yarn shims are broken. The strongest
candidate project is a Go 1.26 application with a Vite front end, Linux and
macOS release targets, a TUI and support for several non-OpenCode agent
sources.

### Decision

Build OpenCode Observatory as a clean-room TypeScript application using a
Next.js App Router web application, strict TypeScript, npm workspaces, a
separate dashboard-owned SQLite database, dedicated read-only source
adapters and a separate ingestion module. The recommended package boundaries
are adapted to that workspace rather than copied mechanically from a template.

### Rationale

This is the only examined strategy that:

- matches the working environment without requiring another language
  toolchain;
- keeps one language and runtime from source access through the interface;
- matches the PRD's engineering direction;
- supports the four coherent product experiences without inheriting a TUI or
  multi-source scope;
- permits local-only operation while keeping future hosted boundaries
  explicit.

### Rejected alternatives

- **Rejected:** fork the Go dashboard wholesale. It cannot be built or tested
  here, does not publish a Windows binary, and carries a much wider product
  scope.
- **Rejected:** use the Go dashboard as a local backend behind a separate
  Next.js front end. This adds a language toolchain, a second runtime and an
  unnecessary local HTTP boundary while inheriting out-of-scope behaviour.
- **Rejected:** base the product on the legacy-only Next.js candidate. It
  misses V2 sessions and would require replacing most of its data and
  security layers.

### Consequences

- The product owns its domain model, adapters, ingestion and analytics code.
- Initial setup is larger than forking, but every layer can be tested against
  the actual source schema and the Windows-first requirement.
- Phase 2 starts with workspace scaffolding and the SQLite spike rather than
  a feature-heavy scaffold.

### Verification

A Windows npm-workspace install, typecheck, unit and integration suite, and
production build must succeed. Module boundaries are checked by code
inspection and tests rather than by directory names alone.

---

## ADR-002 — Do not copy candidate code or assets

**Status:** Accepted — candidate code reuse is Rejected
**Date:** 2026-10-31
**Requirements:** `PRD-RULE-007`, `PRD-VIS-009`, `PRD-STD-005`

### Context

The three candidates are permissively licensed, but each contains behaviour or
scope that conflicts with the product contract: source mutation, an unsafe
network posture, legacy-only schema assumptions, or broad multi-source and AI
features. Their strongest architectural patterns are small and can be
reimplemented independently.

### Decision

No code, SQL, styles, images, generated files or other candidate assets are
copied into the initial implementation. Candidate 3 may be used as an
architectural reference for independently reimplementing proven ideas:
runtime layout detection, read-only DSN behaviour, fork boundary handling,
missing-usage semantics, per-ledger provenance, synthetic fixtures and host
header protection.

### Rationale

A clean-room implementation avoids importing unsafe behaviour and unnecessary
dependency trees. It also keeps the Observatory domain independent from the
candidates' schema, product scope and licence conditions. Observable ideas are
not copied expression and can be validated against this product's own tests.

### Rejected alternatives

- **Rejected:** copy the legacy-only candidate's query layer. It binds to
  retired table and payload assumptions and weakens read-only guarantees.
- **Rejected:** copy the Go dashboard's source-access package. A direct
  translation keeps the tested design intent while avoiding unnecessary
  Apache-2.0 notice and modified-file obligations in the initial build.

### Consequences

- Implementation takes longer than a fork and must earn each reused idea with
  an Observatory-owned test.
- Any future change to copy external code requires a new licence review,
  attribution work, dependency review and an ADR update before the code lands.

### Verification

A source and dependency review confirms that no candidate path, distinctive
source fragment or copied asset entered the repository. Third-party notices
are updated in the same change if the decision changes.

---

## ADR-003 — Read the source through SQLite and use the API only through a GET allowlist

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-DATA-001` through `PRD-DATA-007`, `PRD-SCOPE-002`,
`PRD-SEC-006`, `PRD-SEC-010`, `PRD-AC-001` through `PRD-AC-004`

### Context

The local database contains the per-request usage payload required for
auditable analytics. The installed OpenCode service exposes useful reads and
dangerous mutating operations on the same origin. The service is local and
authenticated, but an unrestricted proxy would expose controls that
Observatory has no reason to offer.

### Decision

Open the OpenCode source database through a read-only SQLite URI, enable
`query_only`, use a bounded busy timeout and retry policy, and never issue
DDL or DML against that connection. Use the local API only as an optional,
authenticated supplement through an explicit GET allowlist. Feature-detect
optional endpoints; never proxy the complete API to the browser.

### Rationale

The database is the authoritative local history for the product's historical
metrics. A narrow read adapter gives the product the fields it needs without
exposing OpenCode's mutation surface. Runtime discovery and optional-endpoint
detection are necessary because the installed 2.2.4 service differs from
current upstream documentation.

### Rejected alternatives

- **Rejected:** query the live source directly for every interface request.
  It lacks checkpoints, derived provenance and a stable response model.
- **Rejected:** open the source with default read-write options and rely only
  on `query_only`. The URI-level read-only guarantee is the primary defence.
- **Rejected:** proxy the OpenCode API. That would expose mutating endpoints
  and blur the product's security boundary.

### Consequences

- Adapters own schema detection and source-specific decoding.
- Busy, locked or changed schemas produce diagnostics rather than silent
  fallback.
- Source database paths and service credentials stay server-side.

### Verification

A write against a read-only scratch fixture is refused; a stopped fixture's
hash and schema remain unchanged across an import; live WAL reads succeed;
and a server test proves no non-allowlisted API route is proxied.

---

## ADR-004 — Store all derived data in a separate dashboard-owned SQLite database

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-DATA-012`, `PRD-DATA-022` through `PRD-DATA-026`,
`PRD-DATA-029` through `PRD-DATA-031`, `PRD-SCOPE-004`, `PRD-REL-001`

### Context

The product needs checkpoints, canonical entities, derived metrics, cost
ledgers, full-text search, summaries and recovery metadata. None of those
belong in the user's OpenCode database, which must remain an untouched record
of work.

### Decision

Use a separate SQLite database for Observatory. It owns its migrations and
metadata. Keep raw imported records and derived tables logically separated,
with documented retention rules and indexes justified by actual query plans.
Deleting or rebuilding this database must never affect the source.

### Rationale

A separate store provides a safe rebuild boundary, transactional ingestion
checkpoints, schema evolution independent of OpenCode, and query performance
control. It also makes source integrity easy to prove.

### Rejected alternatives

- **Rejected:** add Observatory tables or migrations to the source database.
  This directly violates the product's strongest safety rule.
- **Rejected:** keep all state in browser storage. It is not transactional,
  not durable across browsers, unsuitable for large transcript data and
  cannot support incremental ingestion or recovery.
- **Rejected:** use a hosted database in the initial release. It violates the
  local-first requirement and creates privacy and operational dependencies.

### Consequences

- The source adapter and dashboard store have distinct schemas and
  lifecycles.
- Phase 2 defines migration ownership, backup and rebuild guidance.
- Future hosted mode can map the same domain records through an explicit sync
  boundary without changing local ownership.

### Verification

Tests delete and rebuild the dashboard database, compare source fingerprints
before and after, and prove that checkpoints and derived rows commit
atomically.

---

## ADR-005 — Use canonical identity, V2 precedence and explicit deduplication

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-DATA-009` through `PRD-DATA-011`, `PRD-DATA-016`,
`PRD-DATA-017`, `PRD-DATA-027`, `PRD-DATA-028`, `PRD-DATA-030`,
`PRD-DATA-031`, `PRD-VIS-041`

### Context

The audited source has 140 legacy sessions duplicated into V2, eight
legacy-only sessions, 1,085 initially parent-linked V2 sessions across 35
parents, and fork fields that are currently null. A naïve union would double
count. A naïve V2-only rule would silently lose the eight legacy-only
sessions.

### Decision

Use the OpenCode session ID as the base canonical identity and preserve the
source ID and source layout beside it. For overlapping legacy and V2
sessions, V2 is authoritative after migration is complete. Retain all
legacy-only sessions. Keep parent and child sessions individually traceable,
but attribute usage to the request that owns it instead of adding parent and
child aggregates. Exclude copied fork history at a verified fork boundary;
quarantine ambiguous records and report them. Track layout and ownership
provenance on every imported record.

### Rationale

The rule reconciles the observed local data without silently dropping
history. It handles the dominant parent/child relationship without making
parent sessions invisible, and it leaves fork support testable even though
current local rows do not exercise it.

### Rejected alternatives

- **Rejected:** read V2 only after migration. This loses the eight
  legacy-only sessions.
- **Rejected:** read both layouts and deduplicate by title. Titles are not
  stable identities and can collide.
- **Rejected:** sum parent and child session aggregates. This double counts
  work and cost.

### Consequences

- The canonical layer needs explicit ownership and provenance fields.
- Reconciliation reports must distinguish overlap, legacy-only, parent/child
  and ambiguous cases.
- Fork handling remains fixture-tested and explicitly unverified against
  current local source data until a real fork record appears.

### Verification

Synthetic migrated, legacy-only, parent/child and fork fixtures must produce
one canonical record per overlap, retain legacy-only records, preserve source
and layout IDs, and produce request totals that reconcile with session
aggregates where definitions match.

---

## ADR-006 — Maintain six separate cost ledgers

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-COST-001` through `PRD-COST-038`, `PRD-SCOPE-016`

### Context

The source stores an OpenCode-calculated cost on sessions and per-request
payloads. The audited data contains 45 distinct model values but no
authoritative invoice or billing source. A models.dev catalogue exists in the
source, but it is not an effective-dated Observatory rate card. Combining
measured, reported, recorded, estimated and hypothetical amounts into one
number would create a false billing claim.

### Decision

Represent cost as six separate, typed ledger concepts:

1. actual billed spend;
2. provider-reported usage cost;
3. OpenCode-recorded cost;
4. API-equivalent estimate;
5. free or included-usage equivalent;
6. unknown or not comparable.

Each amount stores amount, currency, cost type, provider and model identity,
source, source reference, pricing-catalogue version and effective date where
relevant, calculation timestamp, quality status and measured/reported/
estimated/hypothetical classification. Never combine actual spend and
hypothetical value in a single total. Missing usage is null or unavailable,
never zero.

### Rationale

The six concepts answer materially different questions. Keeping them separate
makes provenance visible, permits defensible historical and current repricing,
and allows the product to remain honest where billing evidence is absent.

### Rejected alternatives

- **Rejected:** one “total cost” field. It conflates measured and hypothetical
  value and encourages false billing claims.
- **Rejected:** treat `session_v2.cost` as actual spend. Its provenance is
  OpenCode's stored or calculated value, not a reconciled invoice.
- **Rejected:** fill missing token or cost fields with zero. That turns an
  unknown into evidence of no usage.

### Consequences

- The schema models the six ledgers now even though only recorded cost and
  estimates can initially be populated.
- Pricing is a versioned product-owned catalogue with effective dates and
  source URLs; the models.dev payload is only an input.
- Current-rate views are the default; historical-rate views are separately
  selectable and never overwrite the original calculation.

### Verification

Unit tests cover every token category, cache pricing, rate-card version,
currency and missing field. UI and schema tests prove that actual and
hypothetical values are never merged and that missing usage is not zero.

---

## ADR-007 — Keep AI optional, local-first by default and evidence-bound

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-AI-001` through `PRD-AI-029`, `PRD-SEC-005`,
`PRD-SEC-006`, `PRD-SEC-013`, `PRD-SEC-014`, `PRD-RULE-012`, `PRD-RULE-013`

### Context

Project intelligence can summarise outcomes, but transcripts are untrusted
private content. Deterministic analytics does not need AI. Remote processing
can disclose source code, paths or secrets and is subject to prompt
injection from historical text.

### Decision

Keep AI disabled until the user configures a model and explicitly requests
processing. Deterministic metrics must not depend on AI. Support swappable
local and remote provider adapters, batch work, budgets, rate limits, caching
and cancellation. Redact likely secrets and configurable sensitive fields
before external processing. Treat transcript content as evidence, never as
instructions, and never execute historical tools or commands. Validate output
against a typed schema and link claims to real source records where possible.
Store model, provider, prompt and schema version, source hashes, generation
time and confidence. Make derived AI data regenerable and deletable without
touching source history.

### Rationale

This satisfies the intelligence requirement without making privacy,
prompt injection or cost optional. It preserves deterministic truth and makes
every inference auditable.

### Rejected alternatives

- **Rejected:** AI by default or an automatic archive-wide backfill. It
  creates undisclosed cost and privacy exposure.
- **Rejected:** send transcript fragments to a remote model without
  redaction and explicit disclosure.
- **Rejected:** let the model execute or follow commands found in history.

### Consequences

- Phase 5 follows the validated data and analytics foundation.
- A local model can be the first configured provider, but there is no hidden
  default.
- Prompt-injection and redaction tests are release requirements.

### Verification

E2E tests prove no remote call occurs without configuration and action.
Schema, evidence-link, prompt-injection and redaction tests prove output
safety and provenance.

---

## ADR-008 — Keep local mode complete and make hosted sync an explicit future boundary

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-HOST-001` through `PRD-HOST-013`, `PRD-VIS-001`,
`PRD-VIS-007`

### Context

A hosted Next.js server cannot read a developer's local SQLite database.
The product is initially local on Windows, but the PRD asks for interfaces
that do not require a rewrite if a hosted mode is later approved.

### Decision

Build and operate local mode as a complete product. Keep domain, storage and
potential sync interfaces separate. A future local collector may explicitly
select, sanitise and synchronise data to a hosted frontend or API. That sync
must be user-initiated, authenticated, encrypted in transit, revocable,
minimal and documented. Hosted storage, tenancy, encryption, retention and
cost require a separate review. No deployment, port forwarding or public
collector exposure occurs in the initial phases.

### Rationale

Designing for hosted mode at the interface boundary preserves future choice
without allowing an impossible hosted-to-local data path or premature cloud
dependency.

### Rejected alternatives

- **Rejected:** deploy the local Next.js server to Vercel and expect it to
  read the PC database.
- **Rejected:** expose a local collector publicly with port forwarding.
- **Rejected:** upload the whole database automatically for future
  convenience.

### Consequences

- The local data store remains authoritative in the initial release.
- Domain records can be serialised for a future sync adapter without
  exposing the source DB path or secrets to browser code.
- Hosted readiness is a Phase 7 assessment, not a deployment.

### Verification

Local operation passes with networking disabled. Security configuration proves
loopback binding and no public listener. Phase 7 documents what a hosted
deployment can and cannot do.

---

## ADR-009 — Adopt a deny-by-default local security posture

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-SEC-001` through `PRD-SEC-014`, `PRD-NFR-010`,
`PRD-RULE-010`, `PRD-RULE-011`, `PRD-RULE-013`

### Context

The product exposes private prompts, file paths, tool output and model
history on a developer machine. A browser-based local server can also be
reached from other software on the same host or through DNS rebinding if it
accepts arbitrary Host headers.

### Decision

Bind the local server to loopback by default. Validate Host headers against
an allowlist and apply Origin checks to state-changing local routes. Keep
source paths, service credentials and provider keys server-side. Send no
telemetry and perform no remote upload by default. Parameterise every SQL
statement. Treat transcript text, titles, filenames and tool output as
untrusted. Never execute historical commands. Bound exports and protect them
against path traversal. Redact secrets from display, logs, exports and AI
payloads. Provide explicit controls to reset only dashboard-owned data.

### Rationale

The data is sensitive and the local service is still an attack surface. The
controls are proportionate to a single-user local app and provide explicit
paths for later hosted review.

### Rejected alternatives

- **Rejected:** bind to all interfaces for convenience. It exposes private
  history without authentication.
- **Rejected:** rely only on CORS. CORS does not prevent non-browser requests
  and does not change the listener address.
- **Rejected:** allow arbitrary export paths. It turns a local read feature
  into a file-disclosure primitive.

### Consequences

- Security controls are implemented with the foundation rather than added
  after the UI.
- The browser receives only data needed for the current view.
- Host and Origin behaviour, export bounds and redaction receive dedicated
  automated tests.

### Verification

Security tests cover write refusal, foreign Host headers, disallowed Origin,
path traversal, oversized export, prompt injection and redaction. Configuration
inspection proves no telemetry or public listener by default.

---

## ADR-010 — Use Next.js, strict TypeScript and the maintained UI stack

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-ARCH-001` through `PRD-ARCH-014`, `PRD-ARCH-010` through `PRD-ARCH-013`, `PRD-VIS-010` through
`PRD-VIS-019`, `PRD-NFR-008` through `PRD-NFR-012`

### Context

The Master Prompt recommends Next.js and TypeScript, Tailwind CSS, shadcn/ui,
Lucide and Recharts or a suitable maintained chart library. The baseline has no
application stack to preserve. The product needs a server-side data boundary,
typed domain code, a premium dark interface and charts backed by real data.

### Decision

Use Next.js App Router, strict TypeScript, Tailwind CSS v4, shadcn/ui where
compatible, Lucide icons and Recharts. Validate data at external boundaries,
keep domain and data logic outside React components, and build a reusable
component layer with explicit design tokens. shadcn/ui compatibility is to be
confirmed during the Phase 3 shell and must not block the data foundation.

### Rationale

This matches the product direction and provides one TypeScript path from
server-side source access to the browser interface. The library choices are
maintained, accessible and suitable for dense developer-tool dashboards.

### Rejected alternatives

- **Rejected:** a Go backend with a Vite SPA, because it adds a runtime and
  toolchain and has no Windows release path.
- **Rejected:** a server-rendered dashboard that queries the source directly
  for every request, because it lacks the accepted ingestion and provenance
  model.

### Consequences

- Server Components and route handlers may read the dashboard store, but the
  source database remains behind adapters and ingestion boundaries.
- Chart data comes from the dashboard store or clearly labelled fixtures.
- Accessibility and responsive states are acceptance criteria, not polish
  added after the feature.

### Verification

Typecheck, lint, unit, integration, production build, browser interaction and
responsive checks run in the applicable phase. Component and chart choices are
reviewed against actual data and empty states.

---

## ADR-011 — Use npm workspaces

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-ARCH-001`, `PRD-ARCH-014`, `PRD-ARCH-010` through `PRD-ARCH-013`, `PRD-NFR-008`

### Context

The repository has no package manager. npm 11.19.0 is installed and works;
the global pnpm and Yarn shims are broken; Bun is absent. The product needs
multiple internal packages without publishing them.

### Decision

Use npm workspaces at the repository root. Pin the Node and npm versions in
project documentation and CI. Commit the npm-generated lockfile and regenerate
it through npm rather than hand-editing it. Do not repair global pnpm or Yarn
shims as part of the product.

### Rationale

npm is the only verified package-manager path on the host, needs no global
repair and supports the planned workspace layout.

### Rejected alternatives

- **Rejected:** pnpm or Yarn until their global shims are repaired and
  verified. Their absence should not block the foundation.
- **Rejected:** a single root application package. It weakens the required
  module ownership boundaries.
- **Rejected:** Bun, which is not installed and adds an unnecessary runtime
  choice.

### Consequences

- The initial install creates one root lockfile and workspace links.
- Dependency and CI changes are expressed through npm commands.
- A future package-manager change requires an ADR because it affects the
  lockfile and every developer workflow.

### Verification

A clean Windows checkout installs with `npm ci`, typechecks and builds using
only npm scripts.

---

## ADR-012 — Select the SQLite client after a Windows `node:sqlite` spike

**Status:** Proposed
**Date:** 2026-10-31
**Requirements:** `PRD-DATA-001` through `PRD-DATA-007`,
`PRD-DATA-022`, `PRD-DATA-029` through `PRD-DATA-031`, `PRD-NFR-009`

### Context

Node 24 includes `node:sqlite` without the old experimental flag. Its API
offers read-only opening, a busy timeout, defensive mode, an authorizer,
change sessions, backup and location reporting. However, the module is still
described as a release candidate in the Node 24 documentation, and the
audited environment has no SQLite CLI or native-build evidence for an
alternative driver.

### Decision

Use Node's built-in `node:sqlite` as the leading candidate, but keep the
client decision **Proposed** until a bounded Phase 2 spike demonstrates on
Windows:

- opening an existing database read-only without creating a missing file;
- enforcing query-only behaviour;
- a bounded busy timeout against a WAL fixture;
- parameterised statements and safe type conversion;
- transaction and checkpoint behaviour;
- migrations and schema inspection;
- an acceptable path to the required full-text search and backup/restore
  behaviour.

The source-access and dashboard-store interfaces must be client-neutral so a
supported npm-native fallback can replace it without changing domain logic.
Do not add a native compiler dependency to the product merely to force the
leading candidate.

### Rationale

A built-in client avoids a native dependency and gives direct control over
read-only and defensive settings. Deferring the commitment avoids making the
whole data foundation depend on an unproven Windows combination.

### Rejected alternatives

- **Rejected:** choose a native-binding SQLite package without checking the
  Windows installation and native-build path.
- **Rejected:** use an in-memory or JSON store for durable source-derived
  data. It cannot provide the required transactional checkpoints and indexed
  queries cleanly.

### Consequences

- The adapter owns a small SQLite capability interface.
- The spike is an early Phase 2 task, before broad database code is written.
- The decision is recorded as accepted or superseded with measured results.

### Verification

A committed spike fixture and test matrix produces repeatable results on
Windows. The product depends only on the capability interface, not on
`node:sqlite` syntax spread through domain packages.

---

## ADR-013 — Build synthetic fixtures for legacy, V2, migrated overlap and relationship edge cases

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-AC-004`, `PRD-AC-006` through `PRD-AC-008`,
`PRD-AC-013`, `PRD-AC-017`, `PRD-AC-020`

### Context

The live source is large, mutable and contains private data, so it cannot be
the only test input. The audit identified layout combinations and edge cases
that must be reproducible without copying user history.

### Decision

Create synthetic SQLite fixtures owned by the project. The minimum matrix
includes legacy-only, V2-only, migrated overlap, legacy-only-after-migration,
parent and many-child relationships, fork boundary, missing usage, partial
token categories, malformed JSON, compaction usage, changed and deleted
records, interrupted ingestion and a busy WAL source. Fixture creation is
deterministic and contains no user transcript or credential.

Use the live source separately for compatibility and reconciliation checks,
never as expected-output data or a mutation target.

### Rationale

Deterministic fixtures turn the audit findings into repeatable failure
tests. They let the team prove dedup, idempotency, resume, missing-value and
security invariants without exposing private data or depending on a changing
2.33 GB database.

### Rejected alternatives

- **Rejected:** test only against the live source. Its counts and contents
  change and expected values would become unreliable.
- **Rejected:** copy sanitised production rows into fixtures. Provenance and
  privacy risks remain, and synthetic cases are clearer.

### Consequences

- Fixture builders live with the test suite and document their schema.
- The same fixtures support source adapters, ingestion, reconciliation and
  security tests.
- Live-source smoke checks remain separate from deterministic test results.

### Verification

CI creates every fixture from scratch, checks the expected counts and runs
the adapter, ingestion, cost and security assertions against it.

---

## ADR-014 — Publish the project under MIT and review every external dependency

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-RULE-007`, `PRD-RULE-014`, `PRD-VIS-009`, `PRD-SCOPE-015`

### Context

The inherited README claimed MIT but the repository had no root licence. The
candidate projects use MIT and Apache-2.0. The product is new and no external
candidate code is copied in Phase 1.

### Decision

Publish the new project under the MIT licence with an explicit repository
contributor copyright line. Maintain a third-party notice inventory for
dependencies and any copied material. Before copying external code or assets,
review the exact licence, preserve required notices, mark modifications where
required and verify dependency licences. Open-source reuse must never bypass
the security or data-correctness review.

### Rationale

A real licence makes the public repository's reuse terms clear. A notice
inventory prevents accidental attribution loss. The decision preserves the
option to adopt compatible external material later without making that
material part of the initial build.

### Rejected alternatives

- **Rejected:** leave the repository without a licence while claiming MIT in
  the README. That is contradictory and gives users no clear grant.
- **Rejected:** copy candidate code now and defer attribution. Attribution
  and modified-file obligations must be resolved in the same change.

### Consequences

- The root `LICENSE` and README agree.
- The owner may confirm the precise copyright-holder wording before release;
  the current wording does not name an individual.
- Third-party notices are updated whenever dependencies or copied material
  change.

### Verification

Repository standards, licence-file presence and the dependency and notice
inventory are checked during review.

---

## ADR-015 — Treat documentation, traceability and phase gates as durable architecture

**Status:** Accepted
**Date:** 2026-10-31
**Requirements:** `PRD-RULE-001`, `PRD-RULE-003`, `PRD-RULE-008`,
`PRD-RULE-010`, `PRD-PHASE-008`, `PRD-PHASE-009`, `PRD-PROC-001` through
`PRD-PROC-009`

### Context

The product contract spans seven phases and many safety-critical clauses. The
inherited template included generic documentation, but its application-specific
workflows were stale. Silent drift between requirements, architecture, code and
reports would make completion claims unreliable.

### Decision

Maintain durable, versioned documentation in `docs/` with a product-specific
README, this ADR, the traceability matrix, roadmap, targets, project document,
system diagram, technology record and changelog. Every approved phase updates
the relevant documents, version line and traceability rows, reports exact
commands and real results, and stops for explicit owner approval. IDs are
stable and never renumbered. Documentation states actual installed versions;
planned technology is labelled planned.

### Rationale

The phase gate is part of the architecture because it protects source safety,
cost correctness and privacy across a long build. Living documents make each
phase reviewable and prevent an agent summary from being mistaken for
evidence.

### Rejected alternatives

- **Rejected:** keep the generic template docs unchanged. They describe a
  different product and application structure.
- **Rejected:** treat a chat report or agent completion notice as durable
  project evidence. The orchestrator must inspect artifacts and command
  output first.
- **Rejected:** start the next phase merely because implementation appears
  complete. Explicit owner approval is required.

### Consequences

- Each future PR changes the version line for the workstream and the matching
  changelog entry.
- `/sync` remains owner-authorised and is not automatically invoked.
- Phase 1 ends with a complete seven-part report and no Phase 2 code.

### Verification

The repository standards and scope checks pass, the traceability validator
finds contiguous non-duplicate IDs and every group, the phase report lists
actual commands and results, and the pull request links the evidence.

---

## Consolidated consequences

The accepted decisions form one coherent architecture:

```text
OpenCode source database
  -> read-only adapter and runtime layout detection
  -> canonical identity, provenance and request-level ownership
  -> checkpointed idempotent ingestion
  -> separate dashboard SQLite database
  -> deterministic analytics and six separate cost ledgers
  -> optional, evidence-bound intelligence
  -> local Next.js interface
  -> optional explicit sanitised sync boundary for a future hosted mode
```

The source remains read-only. The dashboard store is disposable and
rebuildable. The browser receives only scoped local data. AI and cloud
processing remain off until explicitly configured and authorised. Every later
phase must provide evidence against this architecture before its approval
gate.
