# Requirements traceability

Every clause of `PRD.md` and `Master_Prompt.md` mapped to a stable identifier, the
phase that delivers it, the method that proves it and its honest status.

- [Product requirements](#1-product-requirements)
- [Scope](#2-scope-boundary)
- [Cost accounting](#3-cost-accounting)
- [Data and ingestion](#4-data-ingestion-source-safety-schema-compatibility-and-dashboard-database)
- [AI and intelligence](#5-ai-processing-architecture-and-intelligence-output)
- [Reliability](#6-reliability-page-and-operational-resilience)
- [Hosted boundary](#7-hosted-and-vercel-boundary)
- [Security and privacy](#8-security-and-privacy)
- [Non-functional](#9-non-functional-requirements)
- [Architecture](#10-structure-and-module-boundaries)
- [Acceptance criteria](#11-testing-and-acceptance-criteria)
- [Definition of done](#12-definition-of-done)
- [Phase gates](#13-seven-delivery-phase-gates)
- [Per-phase workflow](#14-per-phase-workflow)
- [Open questions](#15-phase-1-open-questions)
- [Final standard](#16-final-product-standard)
- [Phase 1 audit tasks](#17-phase-1-audit-tasks)
- [Coverage self-check](#18-coverage-self-check)
- [How to read and maintain this document](#19-how-to-read-and-maintain-this-document)

## Identifier scheme and status conventions

**Identifier scheme.** `PRD-<GROUP>-<NNN>`. Groups and the clause each one comes
from:

| Group | Source clause |
|---|---|
| `RULE` | Master Prompt section 2, the fourteen non-negotiable rules (the original mapping is retained and missing rules are appended as `RULE-010` through `RULE-014`) |
| `VIS` | PRD sections 1 and 2 (vision, principles and jobs to be done), section 4 visual direction, app shell, navigation and pages A to F |
| `SCOPE` | PRD section 3, in scope and out of scope |
| `COST` | PRD section 5 cost concepts, pricing calculation and provenance, plus section 4 page D |
| `DATA` | PRD section 6 ingestion and storage, plus Master Prompt section 6 data correctness |
| `AI` | PRD section 7 AI architecture and summary schema, section 4 page E, Master Prompt section 8 |
| `REL` | PRD section 4 page F reliability |
| `HOST` | PRD section 8 and Master Prompt section 9, the hosted boundary |
| `SEC` | PRD section 9 security and privacy |
| `NFR` | PRD section 10 non-functional requirements |
| `ARCH` | PRD section 11 recommended structure, plus Master Prompt section 4 module separation |
| `AC` | PRD section 12, testing and acceptance criteria |
| `DOD` | PRD section 12, definition of done |
| `PHASE` | PRD section 13 delivery phases, plus Master Prompt section 11 phase plan |
| `PROC` | Master Prompt section 10, the per-phase workflow |
| `OPEN` | PRD section 14, the Phase 1 open questions |
| `STD` | PRD section 15, final product standard |
| `AUD` | Master Prompt section 3, the Phase 1 audit tasks |

**Status vocabulary.**

| Status | Meaning |
|---|---|
| `Complete` | The requirement is satisfied by work already in this repository and can be checked against the evidence named in the row |
| `Not started` | The requirement is not implemented yet. This is a factual statement: the repository holds no application code |
| `Blocked by Phase 1` | The requirement cannot be implemented until a named Phase 1 finding exists. Used only when the audit genuinely left the prerequisite open |
| `N/A (reason)` | The requirement does not apply to the initial release, with the reason stated |

**Evidence vocabulary.** `Audit` for an audit finding, `Unit` for a unit test,
`Int` for an integration test, `Fixture` for a synthetic fixture, `E2E` for an
end to end test, `Sec` for a security test, `Bench` for a benchmark, `Doc` for a
document review, `UI` for a visual or manual interface review.

**Status basis.** The Phase 1 audit on 2026-10-30 and 2026-10-31. The
repository contains no application source, so every implementation row is
`Not started` as a matter of fact rather than by assumption. Rows whose
prerequisite was a Phase 1 finding now carry the resolved finding in the Notes
column and are `Not started` with that finding recorded, or `Complete` when the
Phase 1 work itself satisfied the row.

## 1. Product requirements

### Master Prompt non-negotiable rules

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-RULE-001 | PRD requirements are extracted into a traceable checklist before coding | 1 | Audit | Complete | This document is the artifact |
| PRD-RULE-002 | Audit the repository, dependencies, source schema, OpenCode version and candidate repositories before any feature code | 1 | Audit | Complete | `docs/phase-1-audit.md` |
| PRD-RULE-003 | Pre-existing work is preserved: status, branches and diffs are inspected before edits and no user change is discarded | 1 to 7 | Audit plus code inspection per phase | Complete | User supplied `PRD.md` and `Master_Prompt.md` are committed here; the two obsolete UI template documents are removed as the user intended |
| PRD-RULE-004 | No fabricated metrics, costs, model capabilities, provider rates or test results; fixtures are labelled as fixtures | 1 to 7 | Unit plus code inspection | Not started | Enforced from Phase 2 with the synthetic fixture suite |
| PRD-RULE-005 | No test is reported as passing unless it was actually run, with the exact command and result recorded | 1 to 7 | Doc, phase report | Not started | The Phase 1 report lists the commands that were actually run |
| PRD-RULE-006 | Unsupported schemas are never silently skipped; the limitation is reported and diagnostics are preserved | 2 | Int plus diagnostic output | Not started | Phase 2 introduces runtime layout detection with explicit diagnostics |
| PRD-RULE-007 | Every candidate repository has its current licence, dependencies and attribution obligations reviewed before any reuse | 1 to 2 | Audit plus code inspection | Complete | Three candidates reviewed in `docs/phase-1-audit.md`; the decision to copy no code is ADR-002 |
| PRD-RULE-008 | Phases proceed sequentially; each phase is reported with evidence and waits for explicit user approval | 2 to 7 | Doc, phase report | Not started | Phase 1 stops at the approval gate |
| PRD-RULE-009 | No deployment to Vercel, no port exposure and no upload of user data without later explicit authorisation | 2 to 7 | Sec plus configuration inspection | Not started | The local server binds loopback only, ADR-009 |
| PRD-RULE-010 | The source database is read-only and all derived data lives in a separate dashboard-owned database | 1 to 7 | Sec plus Unit | Not started | Master Prompt rule 4; ADR-003 and ADR-004 |
| PRD-RULE-011 | Secrets, credentials and private transcripts are never committed or exposed | 1 to 7 | Sec plus code inspection | Not started | Master Prompt rule 8; the source `credential` table is excluded from every adapter |
| PRD-RULE-012 | External AI processing is disabled by default and always identifies where data will be sent | 1 to 7 | Sec plus E2E | Not started | Master Prompt rule 9; ADR-007 |
| PRD-RULE-013 | Historical content is never executed arbitrarily; transcripts, tool output, filenames and session text remain inert data | 1 to 7 | Sec plus code inspection | Not started | Master Prompt rule 10; ADR-007 and ADR-009 |
| PRD-RULE-014 | Actual billed spend, provider-reported usage, OpenCode-recorded cost, API-equivalent estimates and free or included-usage counterfactuals remain distinct | 2 to 7 | Unit plus UI | Not started | Master Prompt rule 11; ADR-006 |

### Vision, principles, visual direction and pages

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-VIS-001 | Source data stays on the developer's machine by default and the app runs with no cloud dependency | 2 to 3 | Int, offline run | Not started | ADR-001 establishes the local TypeScript build; ADR-008 keeps hosted sync an explicit future boundary |
| PRD-VIS-002 | The OpenCode database is never modified by the product | 2 | Unit plus Sec | Not started | The audit already proved a read-only connection works against the live database |
| PRD-VIS-003 | After the initial backfill only new or changed data is imported | 2 | Int, second sync import count | Not started | ADR-004 defines the separate store; ADR-005 defines identity and reconciliation |
| PRD-VIS-004 | Every AI insight links back to the supporting sessions and records | 5 | Int plus E2E | Not started | |
| PRD-VIS-005 | Actual spend, provider reported usage, estimated API equivalent cost and unknown cost are distinguished everywhere | 4 | Unit plus UI | Not started | ADR-006 |
| PRD-VIS-006 | Hundreds to thousands of sessions are handled without a cloud service | 6 | Bench | Not started | The audited database already holds 1,250 sessions and 111,797 transcript rows |
| PRD-VIS-007 | A future hosted version needs no full rewrite and a hosted deployment cannot expose the local database or secrets | 2 interfaces, 7 notes | Doc, ADR plus hosting notes | Not started | ADR-001 preserves the interface boundary and ADR-008 defines future hosting |
| PRD-VIS-008 | Every displayed number has a definition, a source and a provenance | 3 to 4 | UI, tooltip and detail audit | Not started | |
| PRD-VIS-009 | Reused open source code keeps the required attribution and licence notices | 1 to 2 | Audit plus code inspection | N/A (the audit decided to copy no candidate code; if that changes the row reopens) | ADR-002 |
| PRD-VIS-010 | Dark premium theme, restrained blue accent, neutral surfaces, subtle borders, compact readable type, generous spacing, consistent charts, Lucide icons and designed empty, loading and error states | 3 | UI | Not started | The root `design-system/` directory does not exist yet and is created in Phase 3 |
| PRD-VIS-011 | No excessive gradients, oversized cards, decorative clutter or generic template aesthetics | 3 | UI | Not started | |
| PRD-VIS-012 | Responsive layouts for desktop and tablet, with desktop as the primary target | 3, 6 | E2E viewport matrix | Not started | |
| PRD-VIS-013 | The product reads as an engineering observability tool, not a basic admin template | 3 | UI | Not started | Needs human review |
| PRD-VIS-014 | The app shell has a left navigation sidebar with a product mark and the main sections | 3 | UI plus E2E | Not started | |
| PRD-VIS-015 | The top bar provides global search, date range, source and sync status and settings | 3 | UI plus E2E | Not started | |
| PRD-VIS-016 | Project, model and date filters behave consistently across pages | 3 | E2E | Not started | |
| PRD-VIS-017 | Breadcrumbs and clickable drill-down exist from aggregated charts to the contributing records | 3 | E2E | Not started | |
| PRD-VIS-018 | Cost values are labelled actual, reported, estimated or unknown in the interface | 4 | UI | Not started | |
| PRD-VIS-019 | Keyboard navigation works, focus states are visible and contrast is adequate | 3, 7 | E2E plus accessibility audit | Not started | |
| PRD-VIS-020 | The navigation contains Overview, Projects, Sessions, Agent Activity, Models and Costs, Project Intelligence, Files and Changes, Reliability and Data Sources and Settings | 3 to 6 | UI | Not started | |
| PRD-VIS-021 | The Files and Changes section appears only where file change evidence exists in the source | 3 | UI plus Int | Not started | The legacy `part` payload and the V2 `snapshot` field both carry file evidence |
| PRD-VIS-022 | The overview shows totals for sessions, projects, messages and prompts, model requests, tokens and tool calls | 3 | Int plus E2E | Not started | |
| PRD-VIS-023 | The overview shows actual provider spend, reported usage cost, API equivalent cost and free or included usage equivalent as four separate metrics | 4 | Int plus UI | Not started | |
| PRD-VIS-024 | The overview provides time series charts for sessions, tokens, cost and activity | 3 to 4 | Int plus E2E | Not started | |
| PRD-VIS-025 | The overview provides top projects, most active sessions, most used models and costliest sessions | 3 | Int plus E2E | Not started | |
| PRD-VIS-026 | The overview shows the model mix and the input, output and cache token composition | 3 | Int plus E2E | Not started | |
| PRD-VIS-027 | The overview shows sync status, the last successful import and warnings | 3 | UI | Not started | |
| PRD-VIS-028 | The overview applies date range, project, model and provider filters | 3 | E2E | Not started | |
| PRD-VIS-029 | Every metric explains its definition and provenance in a tooltip or detail view | 3 to 4 | UI | Not started | |
| PRD-VIS-030 | Projects are grouped by canonical project or repository path, with worktree relationships where they can be determined | 3 | Int plus UI | Not started | The audit found `workspace_id` NULL on all 1,250 sessions, so worktree relationships come from the `worktree` table and the session directory |
| PRD-VIS-031 | Session search filters by title, project, model, provider, dates and transcript text | 3 metadata, 6 transcript text | E2E | Not started | |
| PRD-VIS-032 | Session detail shows title, status, timestamps, parent and child relationships, messages, model requests, token usage, the cost ledger, tool calls, file references and a generated summary | 3, plus 4 for cost and 5 for summary | UI plus E2E | Not started | |
| PRD-VIS-033 | The transcript viewer distinguishes user messages, assistant messages, tool calls and results, and system metadata | 3 | UI plus Fixture | Not started | The Phase 1 source audit confirmed V2 `user` and `idle` payload key shapes; the remaining row types must be re-confirmed by runtime schema inspection in Phase 2 |
| PRD-VIS-034 | Long histories are paginated or virtualised | 3, 6 | Bench plus E2E | Not started | |
| PRD-VIS-035 | Export of selected metadata or a transcript happens only after an explicit user action | 6 | E2E | Not started | |
| PRD-VIS-036 | Repository paths and project names are never inferred when they are unavailable, and unknown is shown | 3 | Unit plus UI | Not started | |
| PRD-VIS-037 | The agent timeline is rendered by project, session and agent over a selected period | 3 | Int plus E2E | Not started | |
| PRD-VIS-038 | Human prompt markers are visually separate from agent activity intervals | 3 | UI | Not started | The Phase 1 source audit confirmed a V2 `user` payload shape with `text`, `files` and `time.created`; Phase 2 must confirm the row-type contract at runtime |
| PRD-VIS-039 | Tool and model request events are shown when the source records them | 3 | Fixture | Not started | |
| PRD-VIS-040 | Observed activity is visually distinguished from estimated intervals | 3 | UI | Not started | |
| PRD-VIS-041 | Overlapping parent and subagent timelines do not double count totals | 2 to 3 | Unit plus Int | Not started | 1,085 of 1,250 audited sessions carry a `parent_id`, so this is the main dedup risk |
| PRD-VIS-042 | Long gaps and waiting periods appear only where timestamps support them | 3 | Unit | Not started | The Phase 1 source audit confirmed a V2 `idle` payload shape with `outcome` and `time.created`; Phase 2 must confirm the row-type contract at runtime |
| PRD-VIS-043 | Elapsed wall clock time is never labelled CPU time and never assumed to be active work | 3, 6 | Unit plus code inspection | Not started | |
| PRD-VIS-044 | Project intelligence shows a per project summary across sessions with a date range and evidence links | 5 | Int plus E2E | Not started | |
| PRD-VIS-045 | Project intelligence shows goals and requirements discovered in the session history | 5 | Int plus Fixture | Not started | |
| PRD-VIS-046 | Project intelligence shows completed, partially completed, blocked, failed and unverified outcomes | 5 | Int plus Fixture | Not started | |
| PRD-VIS-047 | Project intelligence shows key decisions, files changed, tests mentioned and commits where available | 5 | Int | Not started | |
| PRD-VIS-048 | Project intelligence shows recurring errors, repeated approaches and unresolved follow ups | 5 to 6 | Int | Not started | |
| PRD-VIS-049 | A cross session timeline of project evolution is available | 5 | Int plus E2E | Not started | |
| PRD-VIS-050 | Evidence snippets link to the originating session and message | 5 | Int plus E2E | Not started | |
| PRD-VIS-051 | AI derived views show confidence and the last generated timestamp | 5 | UI | Not started | |
| PRD-VIS-052 | AI generated content is clearly labelled and can be regenerated | 5 | UI plus E2E | Not started | |
| PRD-VIS-053 | Aggregates drill down to the contributing records and tooltips explain metric definitions and provenance | 3 to 4 | E2E | Not started | |
| PRD-VIS-054 | The target user can understand total usage across every project and session | 3 to 4 | Int plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-055 | The target user can identify the projects, sessions and models consuming the most resources | 3 to 4 | Int plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-056 | The target user can compare human prompts with the work that follows them | 3 to 5 | Int plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-057 | The target user can see activity over time, including long-running agent work between prompts | 3 to 6 | Int plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-058 | The target user can understand outcomes, failures, repeated attempts and unfinished work | 3 to 6 | Int plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-059 | The target user can estimate the paid API value of work performed with free models or subscription access | 4 | Unit plus E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-060 | The target user can search old sessions to recover decisions, findings and implementation history | 6 | E2E | Not started | PRD section 2, job to be done |
| PRD-VIS-061 | The target user can compare efficiency between models for similar task categories | 3 to 4 | Int plus E2E | Not started | PRD section 2, job to be done; task-category classification must be explicit and evidence-based |
| PRD-VIS-062 | The target user can keep monitoring new sessions with minimal overhead | 6 | Int plus Bench | Not started | PRD section 2, job to be done |

## 2. Scope boundary

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-SCOPE-001 | The product is a local web application that runs on Windows | 2 to 3, 7 | Int, Windows run | Not started | |
| PRD-SCOPE-002 | The product reads the OpenCode source database read-only and accesses supported APIs only through an explicit GET allowlist, never by proxying the full API | 2 | Int plus Sec | Not started | The audit proved the read-only database path works; the installed API also exposes mutating operations |
| PRD-SCOPE-003 | Legacy and V2 schema inspection and adapters are supported as needed | 2 | Fixture plus Audit | Not started | The audit confirmed both layouts coexist locally: 140 legacy sessions are duplicated into V2 and 8 are legacy only; ADR-005 defines the reconciliation rule |
| PRD-SCOPE-004 | Ingestion writes only into a separate dashboard owned database | 2 | Unit plus Int | Not started | ADR-003 protects the source and ADR-004 owns the dashboard store |
| PRD-SCOPE-005 | Actual spend and API equivalent cost ledgers exist as distinct stores | 4 | Unit plus schema inspection | Not started | ADR-006 |
| PRD-SCOPE-006 | Free or included usage counterfactual pricing is a first class feature | 4 | Unit plus UI | Not started | |
| PRD-SCOPE-007 | AI summaries and insights are optional and under explicit user control | 5 | E2E | Not started | |
| PRD-SCOPE-008 | Local search, filters, export and diagnostics are available | 6 | E2E | Not started | |
| PRD-SCOPE-009 | Tests, backup and restore guidance, security hardening and documentation are delivered | 6 to 7 | Doc plus test run | Not started | Node's SQLite module documents a backup API, but its Windows suitability remains part of the ADR-012 Phase 2 spike |
| PRD-SCOPE-010 | The architecture supports a future hosted frontend paired with a local data agent | 2, 7 | Doc, ADR plus code inspection | Not started | ADR-008 defines the collector and hosted boundary |
| PRD-SCOPE-011 | The product never modifies OpenCode sessions or messages | 1 to 7 | Sec plus Unit | Not started | The source database is the developer's record of work |
| PRD-SCOPE-012 | The product never sends prompts to or controls OpenCode agents | 1 to 7 | Code inspection | Not started | |
| PRD-SCOPE-013 | The product never runs shell commands found in transcripts | 1 to 7 | Sec plus code inspection | Not started | The legacy tool payload carries command text that must stay inert |
| PRD-SCOPE-014 | Session content is never automatically uploaded to a cloud AI provider | 1 to 7 | Sec plus code inspection | Not started | |
| PRD-SCOPE-015 | The initial release is not a multi user SaaS product or a public dashboard | 1 to 7 | Doc | N/A (explicitly out of scope for the initial release) | Revisit only as a separately authorised future product |
| PRD-SCOPE-016 | The product never claims exact actual charges when no authoritative billing source exists | 1 to 7 | Unit plus UI | Not started | No such source exists locally, so the ledger stays empty and values stay unknown |
| PRD-SCOPE-017 | The product does not auto install into the OpenCode desktop interface before extension points are verified | 1 to 7 | Code inspection | N/A (out of scope until the extension points are verified) | The local web page is the initial target |
| PRD-SCOPE-018 | The product never auto processes the entire historical archive with an AI model without explicit approval | 5 | E2E plus UI | Not started | |

## 3. Cost accounting

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-COST-001 | Actual billed spend is populated only from a reconciled provider invoice or billing API charge | 4 | Unit plus Fixture | Not started | No authoritative billing evidence exists on this machine, so the ledger stays empty and the interface shows unknown |
| PRD-COST-002 | Provider reported usage cost is a distinct ledger, clearly identified as provider reported | 4 | Unit plus UI | Not started | |
| PRD-COST-003 | OpenCode recorded cost is a distinct value with its source recorded | 4 | Unit plus Fixture | Not started | The audit confirmed `session_v2.cost` is populated for all 1,250 sessions and per request cost lives in `session_message.data` |
| PRD-COST-004 | API equivalent estimate is computed from token categories against a specified public rate card | 4 | Unit | Not started | |
| PRD-COST-005 | Free or included usage equivalent is the counterfactual API value for requests served with no separately itemised charge | 4 | Unit plus UI | Not started | |
| PRD-COST-006 | Unknown covers amounts with insufficient data and is never treated as zero | 4 | Unit | Not started | The audit found assistant rows with an error and no usage record, which are exactly this case |
| PRD-COST-007 | Token counts are stored per category, model, provider and request | 2 to 4 | Unit plus schema inspection | Not started | The V2 payload carries input, output, reasoning and cache read and write per request |
| PRD-COST-008 | A versioned price catalogue stores model identifier, provider, currency, unit, effective from and to dates, input, output, cache read and cache write rates, long context rules and a source URL | 4 | Unit plus Fixture plus schema inspection | Not started | Default recorded in the audit: current rates with a historical view, and the models.dev catalogue stored by OpenCode is an input rather than the authority |
| PRD-COST-009 | Cost is calculated with the rate card that applies to the selected comparison mode | 4 | Unit | Not started | |
| PRD-COST-010 | A historical rate at time of usage view and a current rate today view are both available | 4 | Unit plus UI | Not started | Default recorded in the audit: current rates today, with the historical view selectable |
| PRD-COST-011 | The calculation breakdown is shown per model, request, session and project | 4 | Int plus UI | Not started | |
| PRD-COST-012 | Providers with similarly named models are not assumed to share rates or tokenisation | 4 | Unit plus code inspection | Not started | The audited database holds 45 distinct model values and 1,218 sessions with a model payload |
| PRD-COST-013 | An uncertain model or provider mapping yields a range or not reliably comparable, never a fabricated precise result | 4 | Unit | Not started | |
| PRD-COST-014 | Local models show API equivalent value only when the user selects a comparable hosted model and rate card, otherwise not applicable or unknown | 4 | Unit plus E2E | Not started | Default recorded in the audit: not applicable until the user selects a comparable rate card |
| PRD-COST-015 | Free tier quota usage distinguishes no separate per token charge to the user from a provider cost that cannot be inferred | 4 | Unit plus UI | Not started | |
| PRD-COST-016 | Subscription use is not assumed economically free: included in subscription and the optional hypothetical API value are reported separately | 4 | Unit plus UI | Not started | |
| PRD-COST-017 | Every cost amount stores an amount, a currency and a cost type | 4 | Unit plus schema inspection | Not started | |
| PRD-COST-018 | Every cost amount stores its provider and model identity | 4 | Unit | Not started | |
| PRD-COST-019 | Every cost amount stores its source: billing API, invoice import, OpenCode field, pricing catalogue or manual import | 4 | Unit | Not started | |
| PRD-COST-020 | Every cost amount stores a source record or reference | 4 | Unit | Not started | |
| PRD-COST-021 | Every cost amount stores the pricing catalogue version and effective date where relevant | 4 | Unit | Not started | |
| PRD-COST-022 | Every cost amount stores a calculation timestamp | 4 | Unit | Not started | |
| PRD-COST-023 | Every cost amount stores a confidence or quality status | 4 | Unit | Not started | |
| PRD-COST-024 | Every cost amount is flagged as measured, provider reported, estimated or hypothetical | 4 | Unit plus UI | Not started | |
| PRD-COST-025 | The product answers what the same work would have cost through the paid API | 4 | Unit plus UI | Not started | |
| PRD-COST-026 | The recorded or actual cost of the original usage is reported next to the counterfactual | 4 | UI | Not started | |
| PRD-COST-027 | The original access is classified as free, included in subscription, promotional, locally hosted or unknown | 4 | Unit plus UI | Not started | |
| PRD-COST-028 | The rate card and the token categories used for the counterfactual are reported | 4 | UI | Not started | |
| PRD-COST-029 | The confidence of the counterfactual estimate is reported | 4 | UI | Not started | |
| PRD-COST-030 | Usage is comparable by model, provider, project and task type | 4 | Int plus E2E | Not started | |
| PRD-COST-031 | The token breakdown covers input, output, reasoning when available, cache reads, cache writes and other provider specific categories | 4 | Unit plus Fixture | Not started | |
| PRD-COST-032 | Actual provider spend appears only from authoritative provider billing data or a clearly labelled provider reported source | 4 | Unit plus code inspection | Not started | |
| PRD-COST-033 | Recorded OpenCode cost is shown as a separate source derived value unless its provenance proves actual billed spend | 4 | Unit | Not started | |
| PRD-COST-034 | API equivalent cost is calculated from token categories and the versioned pricing catalogue | 4 | Unit | Not started | |
| PRD-COST-035 | A free or included usage counterfactual is estimated under a user selected paid API rate card | 4 | Unit plus UI | Not started | |
| PRD-COST-036 | Historical pricing and reprice views exist and never overwrite the original calculation | 4 | Unit | Not started | |
| PRD-COST-037 | Confidence, limitations and missing token categories are shown | 4 | UI | Not started | |
| PRD-COST-038 | Actual spend and hypothetical API equivalent value are never added into a single total spend | 4 | Unit plus UI | Not started | Direct violation test |

## 4. Data ingestion, source safety, schema compatibility and dashboard database

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-DATA-001 | The source database is opened with a read-only SQLite URI and query-only mode | 2 | Unit plus Sec | Not started | The audit proved `mode=ro` plus `query_only=ON` works against the live WAL database |
| PRD-DATA-002 | The product never issues writes, migrations, vacuum operations, deletes or schema changes against the source | 1 to 7 | Sec plus code inspection | Not started | ADR-003 fixes the access boundary; source mutations are prohibited in every phase |
| PRD-DATA-003 | The source location is discovered through documented or configured paths and can be selected explicitly rather than hard-coded | 2 | Int plus UI | Not started | The audit found the default path and the `OPENCODE_DB` override |
| PRD-DATA-004 | Database and WAL state are detected and a consistent read snapshot is used where possible | 2 | Int plus Unit | Not started | The audited source is in WAL mode and is written by a running OpenCode instance |
| PRD-DATA-005 | Locked or busy databases use bounded retries and produce informative diagnostics | 2 | Unit plus Int | Not started | A five-second busy timeout was sufficient in the audit |
| PRD-DATA-006 | The source database remains usable by OpenCode while the dashboard reads it | 2 to 6 | Int | Not started | Concurrent read-only access succeeded during the audit |
| PRD-DATA-007 | The actual source schema is inspected at runtime | 2 | Unit plus Int | Not started | Runtime table and column detection is required; column names are never assumed |
| PRD-DATA-008 | Legacy and V2 source adapters are built from the verified schemas rather than guessed table or column names | 2 | Fixture plus Audit | Not started | The audit verified both layouts and their different payload encodings |
| PRD-DATA-009 | Canonical identifiers exist for sessions, messages, parts, projects, models, providers and tool events | 2 | Unit plus schema inspection | Not started | The canonical domain layer owns these identifiers; source IDs remain separate |
| PRD-DATA-010 | Overlapping legacy and V2 records and parent or subagent activity are deduplicated | 2 | Unit plus Int | Not started | The audit found 140 overlapping sessions, 8 legacy-only sessions and 1,085 parent-linked V2 sessions; ADR-005 defines the rule |
| PRD-DATA-011 | Source identifiers are preserved for traceability | 2 | Unit plus schema inspection | Not started | Source ID and source layout are retained beside the canonical identity |
| PRD-DATA-012 | Migration and version metadata is added only to the dashboard-owned database | 2 | Unit plus schema inspection | Not started | Source migration metadata is read-only evidence and is never changed |
| PRD-DATA-013 | Initial backfill reports progress and can be cancelled | 2 | Int plus E2E | Not started | |
| PRD-DATA-014 | Ingestion uses persistent checkpoints and idempotent upserts | 2 | Unit plus Int | Not started | Running the same import twice must not create duplicate records or costs |
| PRD-DATA-015 | Newly created and updated sessions are detected | 2 | Int | Not started | The incremental path uses verified timestamps and content fingerprints |
| PRD-DATA-016 | Edits and deletions are detected where the source supports them, using tombstones or controlled reconciliation | 2 | Int plus Fixture | Not started | Unsupported deletion semantics produce a diagnostic rather than a silent omission |
| PRD-DATA-017 | Ingestion resumes safely after a crash or restart | 2 | Unit plus Int | Not started | A checkpoint and committed batch boundaries are required |
| PRD-DATA-018 | Routine sync does not reprocess the full transcript on every run | 2 to 6 | Bench plus Int | Not started | |
| PRD-DATA-019 | Content or version hashes invalidate a derived summary only when relevant source data changes | 2, 5 | Unit plus Int | Not started | |
| PRD-DATA-020 | A full analytics rescan or rebuild is available and never touches the source database | 2 | Int plus E2E | Not started | The dashboard-owned database can be deleted and rebuilt independently |
| PRD-DATA-021 | Sync status, last success, imported and updated counts, warnings and error details are visible | 2 to 3 | UI plus Int | Not started | |
| PRD-DATA-022 | All derived data is stored in a separate dashboard-owned SQLite database | 2 | Unit plus schema inspection | Not started | ADR-004 |
| PRD-DATA-023 | Raw source records and derived tables are logically separated in the dashboard database | 2 | Unit plus schema inspection | Not started | ADR-004 |
| PRD-DATA-024 | The dashboard database indexes common filters, transcript search and time-series queries | 2 to 6 | Schema inspection plus Bench | Not started | Query design follows the actual source volume rather than guessed cardinalities |
| PRD-DATA-025 | Duplicate full transcripts are avoided unless required, with references or normalised content retained | 2 | Unit plus schema inspection | Not started | Retention rules are documented with the schema |
| PRD-DATA-026 | Stored source material has documented retention rules | 2, 6 to 7 | Doc plus schema inspection | Not started | |
| PRD-DATA-027 | Calculated totals are compared with OpenCode statistics wherever definitions match, and differences are documented | 2 to 4 | Int plus Doc | Not started | Per-request V2 usage and `session_v2` aggregates are the first reconciliation pair |
| PRD-DATA-028 | Every imported or derived record retains enough source, layout and timing provenance to explain why it was created | 2 | Unit plus schema inspection | Not started | Ambiguous or unsupported records are quarantined and reported rather than silently accepted |
| PRD-DATA-029 | The `credential` table and all authentication material are excluded from every read, transformation, display and export path | 2 to 7 | Sec plus code inspection | Not started | The audited source contains nine credential rows; the adapter must never select that table |
| PRD-DATA-030 | The effective OpenCode CLI and service are discovered from supported locations rather than assumed to resolve on `PATH` | 2 | Int plus fixture | Not started | The host has a 2.2.4 desktop-bundle CLI, a stale 1.14.22 CLI and no `opencode` command on `PATH` |
| PRD-DATA-031 | Missing source values remain null or unavailable in imported records and are never converted to zero | 2 to 4 | Unit plus Int | Not started | Master Prompt section 6; this applies to usage and non-usage fields at the adapter boundary |

## 5. AI processing architecture and intelligence output

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-AI-001 | AI is optional and remains disabled until the user configures a local or remote model | 5 | E2E plus UI | Not started | |
| PRD-AI-002 | Deterministic metrics do not depend on AI | 3 to 4 | Int plus E2E | Not started | Analytics remains fully usable with the intelligence layer absent or disabled |
| PRD-AI-003 | A provider and model adapter interface allows local models and remote APIs to be swapped | 5 | Unit plus code inspection | Not started | |
| PRD-AI-004 | AI work is batched, respects budgets and rate limits, and caches results | 5 | Unit plus Int | Not started | |
| PRD-AI-005 | The product can summarise a selected session, summarise a project and perform an explicitly approved bulk backfill with preview, estimate, budget and cancellation where possible | 5 | E2E | Not started | |
| PRD-AI-006 | Remote bulk operations show an estimated processing cost before execution where that estimate is available | 5 | Unit plus UI | Not started | |
| PRD-AI-007 | Secrets and configurable sensitive paths or fields are redacted before external processing | 5 | Sec plus Unit | Not started | |
| PRD-AI-008 | Transcript content is untrusted evidence; embedded instructions are ignored and historical tools or commands are never executed | 1 to 7 | Sec plus code inspection | Not started | |
| PRD-AI-009 | Summary provenance stores model, provider, prompt and schema version, source session IDs and hashes, generation time and confidence | 5 | Unit plus schema inspection | Not started | |
| PRD-AI-010 | AI output is validated against a typed schema | 5 | Unit plus Fixture | Not started | |
| PRD-AI-011 | AI claims link to supporting source records where the evidence permits | 5 | Int plus E2E | Not started | |
| PRD-AI-012 | Derived AI data can be regenerated or deleted without removing source history | 5 | Int plus E2E | Not started | |
| PRD-AI-013 | AI inferences are never presented as verified facts | 5 | UI plus E2E | Not started | |
| PRD-AI-014 | A summary states the session objective | 5 | Fixture plus E2E | Not started | |
| PRD-AI-015 | A summary states the actions performed | 5 | Fixture plus E2E | Not started | |
| PRD-AI-016 | A summary states the outcome and an explicit outcome status | 5 | Fixture plus E2E | Not started | |
| PRD-AI-017 | A summary carries evidence references | 5 | Int plus E2E | Not started | |
| PRD-AI-018 | A summary identifies referenced files, tests and commits when available | 5 | Fixture plus Int | Not started | |
| PRD-AI-019 | A summary identifies blockers and remaining work | 5 | Fixture plus E2E | Not started | |
| PRD-AI-020 | A summary identifies repeated issues | 5 | Fixture plus Int | Not started | |
| PRD-AI-021 | A summary carries a confidence value | 5 | Unit plus UI | Not started | |
| PRD-AI-022 | A summary carries its model and provider generation metadata | 5 | Unit plus UI | Not started | |
| PRD-AI-023 | AI implementation begins only after the data foundation and deterministic analytics are validated | 5 | Doc plus phase report | Not started | This is the Phase 5 ordering gate |
| PRD-AI-024 | Outcome status explicitly distinguishes verified, unverified and unknown | 5 | Fixture plus UI | Not started | |
| PRD-AI-025 | Project intelligence synthesises across linked sessions | 5 | Int plus E2E | Not started | |
| PRD-AI-026 | Repeated issues and potential duplicated work are detected and labelled as heuristics | 5 to 6 | Int plus Fixture | Not started | |
| PRD-AI-027 | Remote processing requires explicit opt-in, a visible destination disclosure, budgets and redaction | 5 | E2E plus Sec | Not started | |
| PRD-AI-028 | Evidence links resolve to real source session, message or tool records | 5 | Int plus E2E | Not started | |
| PRD-AI-029 | Prompt-injection protections treat transcript text as evidence rather than instructions | 1 to 7 | Sec plus Fixture | Not started | |

## 6. Reliability page and operational resilience

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-REL-001 | Failed or interrupted sessions are identified where observable | 3, 6 | Int plus Fixture | Not started | V2 assistant errors and idle outcomes provide the first evidence |
| PRD-REL-002 | Tool errors are identified where observable | 3, 6 | Int plus Fixture | Not started | Legacy tool state carries error fields; V2 usage is error-aware |
| PRD-REL-003 | Retry patterns are identified where observable | 3, 6 | Int plus Fixture | Not started | The audited assistant payload carries retry metadata |
| PRD-REL-004 | Incomplete work is identified where observable | 3, 6 | Int plus Fixture | Not started | |
| PRD-REL-005 | Errors have categories and trends | 6 | Unit plus Int | Not started | Categories are deterministic and remain distinct from AI interpretation |
| PRD-REL-006 | Potential duplicate work and repeated attempts are explicitly labelled as heuristics | 6 | Unit plus Int plus UI | Not started | |
| PRD-REL-007 | Session outcome distribution includes an explicit unknown or unverified category | 3, 6 | Int plus UI | Not started | |
| PRD-REL-008 | High token use or long duration alone never causes an approach to be labelled wasteful | 3, 6 | Unit plus code inspection | Not started | |

## 7. Hosted and Vercel boundary

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-HOST-001 | The initial local application works fully without Vercel or a hosted account | 2 to 7 | Int plus offline run | Not started | |
| PRD-HOST-002 | The architecture never assumes a hosted server can read the user's local filesystem or SQLite database | 2, 7 | Doc plus code inspection | Not started | ADR-008 |
| PRD-HOST-003 | A future local agent or collector can read local data, own incremental sync and expose a narrowly scoped authenticated API | 2 interfaces, 7 notes | Doc plus code inspection | Not started | |
| PRD-HOST-004 | A future hosted frontend or API consumes only data explicitly selected, sanitised and uploaded or securely synchronised by the user | 7 | Doc plus security review | Not started | |
| PRD-HOST-005 | Cloud storage or a cloud database is used only after a separate privacy, authentication, tenancy, encryption, retention and cost review | 7 | Doc plus security review | Not started | |
| PRD-HOST-006 | Local-only mode remains supported | 2 to 7 | Int plus offline run | Not started | |
| PRD-HOST-007 | The local collector is not exposed through port forwarding or the public internet by default | 2 to 7 | Sec plus configuration inspection | Not started | Loopback binding and Host and Origin checks are required |
| PRD-HOST-008 | Any cloud sync is explicit, opt-in, authenticated, encrypted in transit, revocable and documented | 7 | Sec plus E2E | Not started | |
| PRD-HOST-009 | Provider API keys and local secrets stay out of the browser and source control | 2 to 7 | Sec plus code inspection | Not started | |
| PRD-HOST-010 | Documentation states what does and does not work in a Vercel deployment before deployment is enabled | 7 | Doc | Not started | |
| PRD-HOST-011 | Domain, storage and sync interfaces keep a future hosted version from requiring a full rewrite | 2, 7 | Doc plus code inspection | Not started | |
| PRD-HOST-012 | No Vercel deployment occurs in the initial phases | 1 to 6 | Doc plus configuration inspection | Not started | Deployment requires later explicit authorisation |
| PRD-HOST-013 | A hosted release has a reviewed plan for authentication, tenancy, encryption, secrets, retention, export, deletion and explicit sync | 7 | Doc plus security review | Not started | |

## 8. Security and privacy

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-SEC-001 | The local server binds to loopback by default | 2 to 7 | Sec plus configuration inspection | Not started | |
| PRD-SEC-002 | The product sends no telemetry by default | 2 to 7 | Sec plus configuration inspection | Not started | |
| PRD-SEC-003 | The product performs no remote upload by default | 2 to 7 | Sec plus configuration inspection | Not started | |
| PRD-SEC-004 | Secrets found in transcripts are never displayed or exported | 3, 6 | Sec plus E2E | Not started | |
| PRD-SEC-005 | Likely API keys, passwords, tokens and credentials are redacted from AI payloads and exported logs | 5 to 7 | Sec plus Unit | Not started | |
| PRD-SEC-006 | Session titles, transcripts, filenames and tool output are treated as untrusted input | 2 to 7 | Sec plus code inspection | Not started | |
| PRD-SEC-007 | Session content never causes automatic command execution | 1 to 7 | Sec plus code inspection | Not started | |
| PRD-SEC-008 | Export paths are protected against traversal | 6 to 7 | Sec plus E2E | Not started | |
| PRD-SEC-009 | Export size is bounded | 6 to 7 | Sec plus E2E | Not started | |
| PRD-SEC-010 | Source database paths and credentials remain on the server side | 2 to 7 | Sec plus code inspection | Not started | |
| PRD-SEC-011 | Clear controls reset dashboard-owned derived data | 2, 6 | Int plus E2E | Not started | Reset never touches the source |
| PRD-SEC-012 | Risks of remote AI processing and hosted deployment are documented | 5, 7 | Doc | Not started | |
| PRD-SEC-013 | Security tests cover prompt injection in transcript content | 5 to 7 | Sec | Not started | |
| PRD-SEC-014 | Security tests cover secret redaction | 5 to 7 | Sec | Not started | |

## 9. Non-functional requirements

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-NFR-001 | Local startup is straightforward and documented for Windows | 2 to 3, 7 | Doc plus Windows run | Not started | The Windows setup guide is a release requirement |
| PRD-NFR-002 | Initial import shows progress and is recoverable | 2 | Int plus E2E | Not started | |
| PRD-NFR-003 | Routine incremental sync processes only changed data | 2 to 6 | Int plus Bench | Not started | |
| PRD-NFR-004 | Common overview queries are indexed and responsive for at least several thousand sessions, measured on real data before timing claims | 3, 6 | Bench | Not started | The audited database already exceeds 1,000 sessions |
| PRD-NFR-005 | Large transcripts are paginated or virtualised | 3, 6 | Bench plus E2E | Not started | |
| PRD-NFR-006 | The interface handles empty, loading, partial-data, error and offline states | 3 to 6 | E2E plus UI | Not started | |
| PRD-NFR-007 | The interface provides adequate contrast, visible keyboard focus and semantic controls | 3, 7 | E2E plus accessibility audit | Not started | |
| PRD-NFR-008 | TypeScript strict mode is enabled where practical | 2 | Typecheck | Not started | |
| PRD-NFR-009 | External and data boundaries use runtime schema validation | 2 | Unit plus Int | Not started | |
| PRD-NFR-010 | SQL statements are parameterised | 2 to 7 | Sec plus code inspection | Not started | |
| PRD-NFR-011 | Diagnostics use structured logs | 2 to 6 | Unit plus code inspection | Not started | |
| PRD-NFR-012 | Automated tests cover ingestion, deduplication, costs, AI schema validation and permissions | 2 to 7 | Unit plus Int plus Sec | Not started | |
| PRD-NFR-013 | Demo data is never presented as real usage, and fixture data is clearly labelled | 1 to 7 | Unit plus UI | Not started | |

## 10. Structure and module boundaries

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-ARCH-001 | The application follows an `apps/web` plus focused `packages` workspace after repository adaptation | 2 | Doc plus code inspection | Not started | The recommended PRD structure is used because the baseline has no application architecture |
| PRD-ARCH-002 | A typed domain package owns canonical entities and boundary validation | 2 | Unit plus code inspection | Not started | |
| PRD-ARCH-003 | Source adapters, ingestion and checkpointing are separate modules with explicit interfaces | 2 | Unit plus Int | Not started | |
| PRD-ARCH-004 | Architecture documentation includes a local installation guide | 2 to 3, 7 | Doc | Not started | `docs/local-installation.md` is the planned location |
| PRD-ARCH-005 | Analytics calculations are isolated from source access and interface code | 2 to 4 | Unit plus code inspection | Not started | |
| PRD-ARCH-006 | The versioned pricing catalogue is isolated from display code | 4 | Unit plus code inspection | Not started | |
| PRD-ARCH-007 | Optional AI provider adapters and intelligence logic are isolated from deterministic analytics | 5 | Unit plus code inspection | Not started | |
| PRD-ARCH-008 | Shared interface components and design tokens have an explicit ownership boundary | 3 | Doc plus code inspection | Not started | |
| PRD-ARCH-009 | Security, redaction and diagnostics modules are separated and reusable | 2 to 7 | Unit plus code inspection | Not started | |
| PRD-ARCH-010 | Local runtime data lives under a gitignored `data/` directory | 2 | Doc plus repository inspection | Not started | The baseline `.gitignore` needs the SQLite exclusions added in Phase 2 |
| PRD-ARCH-011 | Architecture, security and pricing-methodology documentation has a defined home | 2 to 7 | Doc | Not started | |
| PRD-ARCH-012 | The repository has a defined test location and test ownership | 2 to 7 | Doc plus test run | Not started | |
| PRD-ARCH-013 | The recommended monorepo or folder structure is adapted rather than forced when the existing architecture is sound | 1 to 7 | Doc plus ADR | Not started | The audit found no application architecture to preserve, so the proposed workspace is accepted |
| PRD-ARCH-014 | npm workspaces provide one package manager and one dependency graph for the application packages | 2 | Doc plus install run | Not started | ADR-011; pnpm and Yarn shims are broken on the audited machine |

## 11. Testing and acceptance criteria

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-AC-001 | Source database hash, schema and relevant counts are verified before and after tests where feasible | 2 to 7 | Unit plus Int | Not started | A full hash is meaningful only when the live source is stopped; the audit used a header fingerprint while it was live |
| PRD-AC-002 | A test proves read-only access prevents writes | 2 | Sec | Not started | The audit demonstrated refusal on a scratch database; the product test repeats it against a fixture |
| PRD-AC-003 | Tests run with OpenCode active and with a busy or WAL database | 2 | Int | Not started | |
| PRD-AC-004 | Fixtures cover legacy, V2 and overlapping migrated data | 2 | Fixture plus Int | Not started | |
| PRD-AC-005 | Session, message and token counts are reconciled with OpenCode statistics where definitions match | 2 to 4 | Int plus Doc | Not started | |
| PRD-AC-006 | Running sync twice creates no duplicate records or costs | 2 | Int | Not started | |
| PRD-AC-007 | Changed sessions update derived data and stale summaries are invalidated | 2, 5 | Int | Not started | |
| PRD-AC-008 | Parent and subagent relationships are validated without double-counting | 2 to 3 | Unit plus Int | Not started | |
| PRD-AC-009 | Cost tests cover token categories, rate-card versions, cache pricing, missing fields and currency handling | 4 | Unit | Not started | |
| PRD-AC-010 | API-equivalent values never appear as actual billed spend | 4 | Unit plus UI | Not started | |
| PRD-AC-011 | Free and included usage is displayed separately from actual charges | 4 | Unit plus UI | Not started | |
| PRD-AC-012 | Every estimate exposes its calculation provenance | 4 | Unit plus schema inspection | Not started | |
| PRD-AC-013 | Missing usage never defaults to zero | 2 to 4 | Unit | Not started | |
| PRD-AC-014 | AI output is validated against a schema | 5 | Unit plus Fixture | Not started | |
| PRD-AC-015 | AI evidence links resolve to real source records | 5 | Int plus E2E | Not started | |
| PRD-AC-016 | AI summaries include uncertain and unverified outcomes | 5 | Fixture plus UI | Not started | |
| PRD-AC-017 | Tests cover transcript prompt injection and secret redaction | 5 to 7 | Sec | Not started | |
| PRD-AC-018 | Remote AI processing cannot run without explicit configuration and user action | 5 | E2E plus Sec | Not started | |
| PRD-AC-019 | Tests cover filters, drill-down, global search, keyboard access, error states and responsive layouts | 3 to 7 | E2E | Not started | |
| PRD-AC-020 | Tests cover restart and resume, interrupted import, a corrupted dashboard cache and a full rebuild | 2, 6 | Int | Not started | |
| PRD-AC-021 | Local operation has no cloud dependency | 2 to 7 | Int plus offline run | Not started | |
| PRD-AC-022 | The local server does not listen on a public interface by default | 2 to 7 | Sec plus configuration inspection | Not started | |

## 12. Definition of done

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-DOD-001 | The source database remains untouched | 2 to 7 | Sec plus Int | Not started | |
| PRD-DOD-002 | Historical ingestion is reproducible and deduplicated | 2 | Int | Not started | |
| PRD-DOD-003 | Cost measures are clearly separated and auditable | 4 | Unit plus UI | Not started | |
| PRD-DOD-004 | The four main dashboard experiences are complete | 3 to 5 | E2E plus UI | Not started | Overview, explorer, timeline and project intelligence |
| PRD-DOD-005 | AI insights are evidence-backed and optional | 5 | Int plus E2E | Not started | |
| PRD-DOD-006 | Automated tests pass | 2 to 7 | Test run | Not started | Only a command that was actually run may be reported as passing |
| PRD-DOD-007 | Windows installation and recovery documentation is complete | 7 | Doc plus Windows run | Not started | |
| PRD-DOD-008 | Known limitations are documented honestly | 1 to 7 | Doc plus phase report | Not started | This document and each phase report are part of that record |

## 13. Seven delivery phase gates

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-PHASE-001 | Phase 1 performs the repository, candidate and source-data audit and records architecture decisions without feature implementation | 1 | Audit plus Doc | Complete | `docs/phase-1-audit.md` and `docs/architecture-decision-record.md` |
| PRD-PHASE-002 | Phase 2 establishes the local app shell and foundation, source adapters, separate database and read-only incremental ingestion | 2 | Int plus phase report | Not started | Requires explicit approval to begin; the shell is a minimal diagnostics surface, while the premium product shell begins in Phase 3 |
| PRD-PHASE-003 | Phase 3 delivers the premium app shell, overview, project and session explorer, activity timeline and deterministic analytics | 3 | E2E plus phase report | Not started | |
| PRD-PHASE-004 | Phase 4 delivers provenance-aware cost ledgers and free or included-usage API-equivalent counterfactuals | 4 | Unit plus E2E plus phase report | Not started | |
| PRD-PHASE-005 | Phase 5 delivers optional summaries, outcomes and cross-session intelligence | 5 | E2E plus phase report | Not started | |
| PRD-PHASE-006 | Phase 6 delivers incremental monitoring, diagnostics, recovery, search and performance work | 6 | Int plus Bench plus phase report | Not started | |
| PRD-PHASE-007 | Phase 7 delivers security review, acceptance testing, Windows setup and future-hosting notes | 7 | Sec plus test run plus Doc | Not started | |
| PRD-PHASE-008 | Every phase has a written summary, tests and known issues | 1 to 7 | Doc plus phase report | Not started | The Phase 1 exit report is the first instance |
| PRD-PHASE-009 | No next phase starts before the orchestrator reports completion and the user explicitly approves it | 1 to 7 | Doc plus phase report | Complete | Phase 1 stops at this approval gate; no Phase 2 work has started |

## 14. Per-phase workflow

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-PROC-001 | Each approved phase re-reads the relevant PRD requirements and the previous phase report | 2 to 7 | Doc plus phase report | Not started | |
| PRD-PROC-002 | Each approved phase defines a small implementation plan and acceptance tests | 2 to 7 | Doc plus phase report | Not started | |
| PRD-PROC-003 | Existing code is inspected before it is modified | 2 to 7 | Audit plus code inspection | Not started | |
| PRD-PROC-004 | Each approved phase implements the smallest coherent vertical slice | 2 to 7 | Doc plus code inspection | Not started | |
| PRD-PROC-005 | Relevant unit, integration and end-to-end tests, lint, typecheck and build are run where available | 2 to 7 | Test run | Not started | |
| PRD-PROC-006 | Diffs are inspected and source-database access is confirmed read-only | 2 to 7 | Sec plus code inspection | Not started | |
| PRD-PROC-007 | Documentation and requirements traceability are updated | 2 to 7 | Doc | Not started | |
| PRD-PROC-008 | Each phase reports changed files, exact commands and real results, limitations and the next phase | 1 to 7 | Doc plus phase report | Complete | The Phase 1 exit report records its own evidence and limitations |
| PRD-PROC-009 | Each phase stops and waits for explicit approval | 1 to 7 | Doc plus phase report | Complete | Approval is pending for Phase 2 |

## 15. Phase 1 open questions

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-OPEN-001 | Resolve the exact OpenCode version and schema in use | 1 | Audit | Complete | OpenCode 2.2.4; 30 tables; V2 plus a completed-migration legacy snapshot |
| PRD-OPEN-002 | Identify the providers and models in use and the available billing evidence | 1 | Audit | Complete | 45 distinct model values are present; no authoritative invoice or billing source was found locally, so actual and provider-reported ledgers remain unknown |
| PRD-OPEN-003 | Choose whether pricing defaults to current or historical rates | 1 | Audit plus ADR | Complete | Default to current rates today with a separately selectable historical-rate view; never overwrite the original calculation |
| PRD-OPEN-004 | Decide how local models map to a comparable hosted model | 1 | Audit plus ADR | Complete | Mark local-model value not applicable or unknown until the user explicitly selects a comparable hosted model and rate card |
| PRD-OPEN-005 | Decide whether AI summaries default to a local model or remain disabled until configured | 1 | Audit plus ADR | Complete | Remain disabled until a model is configured and the user requests processing |
| PRD-OPEN-006 | Decide which data may later be selected for hosted synchronisation | 1 | Audit plus ADR | Complete | No data is selected now; any future sync is explicit, opt-in, authenticated, encrypted, revocable and separately reviewed |
| PRD-OPEN-007 | Decide whether an existing repository can be reused safely and efficiently | 1 | Audit plus ADR | Complete | Clean-room TypeScript build; no candidate code is copied, and candidate 3 is used only as an architectural reference |

## 16. Final product standard

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-STD-001 | The product is a real usable application rather than a static mockup | 2 to 7 | E2E plus phase report | Not started | |
| PRD-STD-002 | Working code replaces placeholders | 2 to 7 | Code inspection plus phase report | Not started | |
| PRD-STD-003 | Usage data is never fabricated | 1 to 7 | Unit plus UI | Not started | |
| PRD-STD-004 | Unsupported schemas are never silently skipped | 2 | Int plus diagnostic output | Not started | |
| PRD-STD-005 | Completion is never claimed without evidence | 1 to 7 | Doc plus test run | Not started | |
| PRD-STD-006 | The smallest well-tested implementation is preferred and every approximation is documented | 1 to 7 | Doc plus phase report | Not started | |

## 17. Phase 1 audit tasks

| ID | Requirement | Phase | Evidence | Status | Notes |
|---|---|---|---|---|---|
| PRD-AUD-001 | Audit the repository, dependencies, scripts, architecture, git state, OpenCode version and runtime environment | 1 | Audit | Complete | Repository and environment section of `docs/phase-1-audit.md` |
| PRD-AUD-002 | Evaluate all three candidate repositories for licence, maintenance, architecture, schema, features, ingestion, costs, security, tests and reuse obligations | 1 | Audit | Complete | All three evaluations are in `docs/phase-1-audit.md` |
| PRD-AUD-003 | Locate and inspect the source database read-only, including tables, columns, indexes, migrations, WAL state, counts, layouts, duplicates and relationships | 1 | Audit | Complete | Aggregate-only read-only audit; no transcript content or credentials were printed |
| PRD-AUD-004 | Publish the audit, architecture decision record and requirements traceability without implementing features | 1 | Doc | Complete | The three Phase 1 deliverables are the only substantive product artefacts in this phase |
| PRD-AUD-005 | End Phase 1 with the seven-part exit report and wait for approval | 1 | Doc | Complete | The exit report is the final section of `docs/phase-1-audit.md` |

## 18. Coverage self-check

| Group | Rows | Source coverage | Phase 1 resolution |
|---|---:|---|---|
| `RULE` | 14 | Master Prompt section 2 | 4 of 14 complete; enforcement rows not started |
| `VIS` | 62 | PRD sections 1, 2 and 4 | Not started |
| `SCOPE` | 18 | PRD section 3 | Not started, except the explicitly out-of-scope rows |
| `COST` | 38 | PRD section 5 and section 4 Page D | Not started |
| `DATA` | 31 | PRD section 6 and Master Prompt section 6 | Not started |
| `AI` | 29 | PRD section 7, section 4 Page E and Master Prompt section 8 | Not started |
| `REL` | 8 | PRD section 4 Page F | Not started |
| `HOST` | 13 | PRD section 8 and Master Prompt section 9 | Not started |
| `SEC` | 14 | PRD section 9 | Not started |
| `NFR` | 13 | PRD section 10 | Not started |
| `ARCH` | 14 | PRD section 11 and Master Prompt section 4 | Not started |
| `AC` | 22 | PRD section 12 | Not started |
| `DOD` | 8 | PRD section 12 definition of done | Not started |
| `PHASE` | 9 | PRD section 13 and Master Prompt section 11 | Phase 1 complete; later phases await approval |
| `PROC` | 9 | Master Prompt section 10 | Phase 1 reporting complete; later phases await approval |
| `OPEN` | 7 | PRD section 14 | Complete |
| `STD` | 6 | PRD section 15 | Not started |
| `AUD` | 5 | Master Prompt section 3 | Complete |

**Coverage result.** All 18 groups are present, every group begins at `001`, and each
sequence is contiguous. The fourteen Master Prompt rules, the nine PRD jobs to
be done, the seven delivery gates, eight definition-of-done clauses, 22
acceptance criteria, six cost concepts, 13 hosted-boundary clauses, 14 security
clauses and all seven Phase 1 open questions are represented individually
rather than collapsed into vague summaries.

## 19. How to read and maintain this document

- **Stable IDs.** `PRD-<GROUP>-<NNN>` identifiers are permanent. Once published,
  a requirement is never renumbered. If wording changes, edit the requirement
  text and note the change; if a requirement is withdrawn, retain its row and
  mark it `N/A (reason)` rather than reusing the identifier.
- **Append only within a group.** New clauses receive the next unused number in
  their group. Existing rows are updated in place with their evidence and status.
- **Status changes require evidence.** `Complete` means the named evidence was
  actually produced and checked. `Not started` means no implementation exists.
  `Blocked by Phase 1` is reserved for a genuine unresolved prerequisite and
  must name it. `N/A (reason)` requires an explicit reason.
- **Phase ownership.** The Phase column identifies the phase that delivers the
  requirement, not the phase in which it was first mentioned. A row may span
  phases when a later phase adds a necessary surface.
- **Evidence must be runnable.** Every future status change names a command,
  test, artifact or manual inspection that can be repeated. A summary from an
  agent is not evidence until the orchestrator has checked the artifact or
  command output.
- **Current Phase 1 conclusion.** The audit resolved all seven open questions
  and produced the architecture decisions. Phase 2 has not started and requires
  explicit approval.
