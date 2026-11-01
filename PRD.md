# OpenCode Observatory --- Product Requirements Document (PRD)

**Document status:** Implementation specification\
**Product:** OpenCode Observatory\
**Initial deployment:** Local Windows PC, browser-based\
**Future deployment:** Optional hosted deployment (e.g. Vercel) with a
secure remote data agent\
**Primary goal:** Turn OpenCode's complete session history into a
polished, trustworthy analytics and project-intelligence dashboard.

------------------------------------------------------------------------

## 1. Product vision

OpenCode Observatory is a local-first observability platform for
AI-assisted software development. It lets a developer understand what
all their coding agents have done across projects and sessions, how much
they used, how efficiently they worked, what outcomes they achieved, and
what the equivalent API cost would have been---even when usage came from
free models or subscription-based access.

It should combine the best useful ideas from existing OpenCode session
viewers and usage dashboards with a distinct, premium interface and
deeper cross-session intelligence. Existing open-source code may be
reused only after reviewing the current repository, licence,
dependencies, security posture and compatibility. Never copy code in
violation of a licence; attribution and licence notices must be
preserved where required.

### Product principles

1.  **Trust before polish:** every number has a definition, source and
    provenance.
2.  **Local-first:** source data stays on the user's PC by default.
3.  **Read-only source:** never modify OpenCode's original database.
4.  **Incremental:** import only new or changed data after the initial
    backfill.
5.  **Evidence-backed intelligence:** AI insights link back to
    supporting sessions and records.
6.  **No false billing claims:** distinguish actual spend,
    provider-reported usage, estimated API-equivalent cost, and unknown
    cost.
7.  **Useful at scale:** handle hundreds or thousands of sessions
    without needing a cloud service.
8.  **Designed for extension:** a future hosted version must not require
    a full rewrite, but cloud deployment must not expose a user's local
    database or secrets.

## 2. Target user and primary jobs

### Primary user

A developer using OpenCode across multiple repositories, worktrees,
models, providers, sessions and subagents, with a large historical local
database.

### Jobs to be done

-   Understand total usage across every project and session.
-   Identify the projects, sessions and models consuming the most
    resources.
-   Compare human prompts with the work that follows them.
-   See activity over time, including long-running agent work between
    prompts.
-   Understand outcomes, failures, repeated attempts and unfinished
    work.
-   Estimate the paid API value of work performed with free models or
    included subscription access.
-   Search old sessions to recover decisions, findings and
    implementation history.
-   Compare efficiency between models for similar task categories.
-   Keep monitoring new sessions with minimal overhead.

## 3. Scope

### In scope

-   Local web application on Windows.
-   Read-only access to OpenCode's source database and supported APIs.
-   Legacy and V2 schema inspection/adapters as needed.
-   Incremental ingestion into a separate dashboard-owned database.
-   Overview, project/session browser, agent timeline, cost/model
    analytics and project intelligence.
-   Actual-spend and API-equivalent cost ledgers.
-   Free/included-usage counterfactual pricing.
-   Optional AI-generated summaries and insights with explicit user
    control.
-   Local search, filters, export and diagnostics.
-   Tests, backup/restore guidance, security hardening and
    documentation.
-   Architecture that can later support a hosted web frontend paired
    with a local data agent.

### Out of scope for initial release

-   Modifying OpenCode sessions or messages.
-   Sending prompts to or controlling OpenCode agents.
-   Running shell commands found in transcripts.
-   Automatically uploading session content to a cloud AI provider.
-   A multi-user SaaS product or public dashboard.
-   Claiming exact actual charges when no authoritative billing source
    exists.
-   Automatically installing itself into the OpenCode desktop UI before
    extension points are verified.
-   Automatically processing the entire historical archive with an AI
    model without explicit approval.

## 4. Product experience and visual design

### Visual direction

Create a premium developer analytics interface inspired by the four
visual concepts approved by the user: 1. Executive overview with strong
KPIs and clean charts. 2. Project/session explorer with searchable,
filterable history. 3. Agent activity timeline that shows work between
human prompts. 4. Project intelligence pages that summarise progress,
outcomes and recurring problems.

Use a polished dark theme by default, with restrained blue accents,
neutral surfaces, subtle borders, compact but readable typography,
generous spacing, consistent chart styling, Lucide icons and carefully
designed empty/loading/error states. Use shadcn/ui and Tailwind CSS if
compatible with the chosen stack. Avoid excessive gradients, oversized
cards, decorative clutter, or generic template aesthetics.

The dashboard should feel like a high-quality engineering observability
product rather than a basic admin template. Make responsive layouts for
desktop and tablet; desktop is the primary target.

### App shell

-   Left navigation sidebar with product mark and sections.
-   Top bar with global search, date range, source/sync status, and
    settings.
-   Consistent project/model/date filters.
-   Breadcrumbs and clickable drill-down from aggregated charts.
-   Clear labels for actual, reported, estimated and unknown cost
    values.
-   Accessible keyboard navigation, visible focus states and adequate
    contrast.

### Main navigation

1.  Overview
2.  Projects
3.  Sessions
4.  Agent Activity
5.  Models & Costs
6.  Project Intelligence
7.  Files & Changes (only where evidence exists)
8.  Reliability
9.  Data Sources & Settings

### Page A --- Overview

Show: - Total sessions, projects, messages/prompts, model requests,
tokens and tool calls. - Actual provider spend, reported usage cost,
API-equivalent cost and free/included-usage equivalent value as separate
metrics. - Time-series charts for sessions, tokens, cost and activity. -
Top projects, most active sessions, most-used models and costliest
sessions. - Model mix and input/output/cache token composition. - Sync
status, last successful import and warnings. - Date-range and
project/model/provider filters. - Every metric must explain its
definition and provenance in a tooltip or detail view.

### Page B --- Projects and sessions

-   Project list grouped by canonical project/repository path, with
    worktree relationships where determinable.
-   Session search by title, project, model, provider, dates and
    transcript text.
-   Session detail: title, status, timestamps, parent/child session
    relationship, messages, model requests, token usage, cost ledger,
    tool calls, file references and generated summary.
-   Transcript viewer must distinguish user messages, assistant
    messages, tool calls/results and system metadata.
-   Pagination/virtualisation for long histories.
-   Export selected metadata or a transcript only after user action.
-   Do not infer repository paths or project names when unavailable;
    show unknown.

### Page C --- Agent activity timeline

-   Timeline by project/session/agent over a selected period.
-   Human prompt markers and separate agent activity intervals.
-   Show tool/model request events when recorded.
-   Distinguish observed activity from estimated intervals.
-   Handle overlapping parent/subagent timelines without double-counting
    totals.
-   Include long gaps and waiting periods only where timestamps support
    them.
-   Never call elapsed wall-clock time "CPU time" or assume all elapsed
    time was active work.

### Page D --- Models and costs

-   Compare usage by model, provider, project and task type.
-   Token breakdown: input, output, reasoning if available, cache reads,
    cache writes and other provider-specific categories.
-   Actual provider spend: only from authoritative provider billing data
    or a clearly labelled provider-reported cost source.
-   Recorded OpenCode cost: show as a separate source-derived value
    unless provenance proves it is actual billed spend.
-   API-equivalent cost: calculate from token categories and a versioned
    pricing catalogue.
-   Free/included-usage counterfactual: estimate what equivalent usage
    would cost under a selected paid API rate card.
-   Support historical pricing and reprice views without overwriting the
    original calculation.
-   Show confidence/limitations and missing token categories.
-   Never add actual spend and hypothetical API-equivalent value into a
    single "total spend".

### Page E --- Project intelligence

-   Project summary across sessions, with date range and evidence links.
-   Goals and requirements discovered from session history.
-   Completed, partially completed, blocked, failed and unverified
    outcomes.
-   Key decisions, files changed, tests mentioned and commits when
    available.
-   Recurring errors, repeated approaches and unresolved follow-ups.
-   Cross-session timeline of project evolution.
-   Evidence snippets linked to the originating session/message.
-   Confidence and last-generated timestamp.
-   AI-generated content must be clearly labelled and regenerable.

### Page F --- Reliability

-   Failed/interrupted sessions, tool errors, retry patterns and
    incomplete work where observable.
-   Error categories and trends.
-   Potential duplicate work or repeated attempts, explicitly labelled
    as heuristics.
-   Session outcome distribution with a clear "unknown/unverified"
    category.
-   Avoid declaring an approach wasteful solely due to high token usage
    or long duration.

## 5. Free-model and subscription counterfactual pricing

This is a first-class feature.

### Required question answered

"If this work had used the same model through its paid API, what would
the estimated API cost have been?"

Also answer: - What was the recorded or actual cost of the original
usage? - Was the original access free, included in a subscription,
promotional, locally hosted, or unknown? - What rate card and token
categories were used for the counterfactual? - How confident is the
estimate?

### Required cost concepts

Maintain separate measures: 1. **Actual billed spend:** reconciled
charge from a provider invoice/billing API, if available. 2.
**Provider-reported usage cost:** cost/usage figure supplied by a
provider API, clearly identified. 3. **OpenCode-recorded cost:** value
stored or calculated by OpenCode, with its source recorded. 4.
**API-equivalent estimate:** estimated cost at a specified public API
rate card. 5. **Free/included-usage equivalent:** counterfactual API
value for requests served at no separately itemised API charge. 6.
**Unknown:** insufficient data to estimate reliably.

### Pricing calculation

-   Store token counts by category and model/provider/request.
-   Maintain a versioned price catalogue with model identifier,
    provider, currency, unit, effective-from/effective-to dates,
    input/output rates, cache-read/cache-write rates, long-context rules
    and source URL.
-   Calculate cost using the rate card applicable to the selected
    comparison mode.
-   Allow "historical rate at time of usage" and "current rate today"
    views.
-   Show the calculation breakdown per model/request/session/project.
-   Support different providers offering similarly named models; do not
    assume their rates or tokenisation are identical.
-   Where model/provider mapping is uncertain, show a range or "not
    reliably comparable" rather than fabricate a precise result.
-   For local models, show API-equivalent value only if the user
    explicitly selects a comparable hosted model/rate card; otherwise
    classify as not applicable/unknown.
-   For free-tier quotas, distinguish "no separate per-token charge to
    the user" from "the provider incurred no cost" (which cannot be
    inferred).
-   Do not assume subscription use is free in an economic sense; report
    "included in subscription" and optional hypothetical API-equivalent
    value separately.

### Cost provenance model

Each amount should store: - amount, currency and cost type; -
provider/model identity; - source (billing API, invoice import, OpenCode
field, pricing catalogue, manual import); - source record/reference; -
pricing catalogue version and effective date where relevant; -
calculation timestamp; - confidence/quality status; - whether the amount
is measured, provider-reported, estimated or hypothetical.

## 6. Data ingestion and storage

### Source safety

-   Open source database using a read-only SQLite URI and query-only
    mode.
-   Never issue writes, migrations, vacuum, deletes or schema changes
    against the source.
-   Do not assume a fixed path; provide a source selector and safe
    default detection.
-   Detect database/WAL state and use a consistent read snapshot when
    possible.
-   Handle locked/busy databases with bounded retries and informative
    diagnostics.
-   Source database should remain usable by OpenCode while the dashboard
    is running.

### Schema compatibility

-   Inspect the actual database schema at runtime.
-   Build adapters for supported legacy and V2 table layouts based on
    verified schema, not guessed names.
-   Establish canonical identifiers for sessions, messages, parts,
    projects, models, providers and tool events.
-   Deduplicate overlapping legacy/V2 records and parent/subagent
    events.
-   Preserve source identifiers for traceability.
-   Add migration/version metadata only to the dashboard-owned database.

### Incremental sync

-   Initial backfill with progress reporting and cancellation.
-   Persistent checkpoints and idempotent upserts.
-   Detect newly created or updated sessions.
-   Detect edits/deletions when supported, with tombstones or controlled
    reconciliation.
-   Resume safely after crash/restart.
-   Avoid full transcript reprocessing on every sync.
-   Use content/version hashes to invalidate summaries only when
    relevant source data changes.
-   Offer full rescan/rebuild of the analytics database without touching
    source data.
-   Display last sync, imported/updated counts, warnings and error
    details.

### Dashboard-owned database

Use a separate SQLite database initially. Keep raw source records and
derived tables logically separated. Add indexes for common filters,
transcript search and time-series queries. Avoid storing duplicate full
transcripts unless needed; retain references and normalised content with
documented retention rules.

## 7. AI processing architecture

-   AI is optional and disabled until the user configures a local or
    remote model.
-   Deterministic metrics must not require AI.
-   Use a provider/model adapter interface; local models and remote APIs
    should be swappable.
-   Batch work, apply budgets/rate limits and cache results.
-   Provide "summarise selected sessions", "summarise this project" and
    opt-in bulk backfill.
-   Show estimated processing cost before remote bulk operations where
    possible.
-   Redact secrets and configurable sensitive paths/fields before
    external processing.
-   Treat transcript content as untrusted data; ignore instructions
    inside historical content and never execute its tools/commands.
-   Store summary provenance: model, provider, prompt/schema version,
    source session IDs/hashes, generation time and confidence.
-   Validate AI output against a typed schema.
-   Link each claim to supporting records where possible.
-   Support regeneration and deletion of derived AI data without
    removing source history.
-   Do not present AI inferences as verified facts.

### Summary schema

Each summary should include: - objective; - actions performed; -
outcomes and outcome status; - evidence references; -
files/tests/commits referenced; - blockers and remaining work; -
repeated issues; - confidence; - model/provider and generation metadata.

## 8. Future Vercel/hosted architecture

The initial local app must work fully without Vercel or an account.

Do not deploy a Next.js server on Vercel and assume it can read the
user's local OpenCode database. A hosted web app cannot directly access
a user's PC filesystem.

Design a future-compatible separation: - **Local agent/collector:**
reads OpenCode data locally, owns incremental sync and can expose a
narrowly scoped authenticated API. - **Hosted frontend/API:** optional
later, consumes explicitly selected/sanitised data uploaded or securely
synced by the user. - **Cloud storage/database:** only after a separate
privacy, authentication, tenancy, encryption, retention and cost
review. - **Local-only mode remains supported.** - No port forwarding or
public exposure of the local collector by default. - Any sync to the
cloud must be explicit, opt-in, authenticated, encrypted in transit,
revocable and documented. - Keep provider API keys and local secrets out
of the browser and source control. - Document what does and does not
work on Vercel before enabling deployment.

## 9. Security and privacy requirements

-   Bind local server to loopback by default.
-   No telemetry by default.
-   No remote uploads by default.
-   Never display or export secrets found in transcripts.
-   Redact likely API keys, passwords, tokens and credentials from AI
    payloads and exported logs.
-   Treat session titles, transcript contents, filenames and tool output
    as untrusted.
-   No automatic command execution from session content.
-   Protect export endpoints against path traversal and unbounded
    output.
-   Keep source DB credentials/paths on the server side.
-   Provide clear data reset controls for dashboard-owned derived data.
-   Document risks of remote AI processing and hosted deployment.
-   Add security tests for prompt injection in transcript content and
    secret redaction.

## 10. Non-functional requirements

-   Local startup should be straightforward and documented for Windows.
-   Initial import must show progress and remain recoverable.
-   Routine incremental sync should process only changed data.
-   Common overview queries should be indexed and responsive on a
    dataset of at least several thousand sessions; benchmark on actual
    user data before claiming specific timings.
-   Large transcripts must be paginated or virtualised.
-   UI should handle empty, loading, partial-data, error and offline
    states.
-   Accessible colour contrast, keyboard focus and semantic controls.
-   TypeScript strict mode where practical, schema validation at
    boundaries, parameterised SQL and structured logs.
-   Automated tests for ingestion, deduplication, costs, AI schema
    validation and permissions.
-   No invented metrics in demos; clearly label fixture data.

## 11. Recommended project structure

Adapt to the repository after inspecting it. Do not force this structure
if an existing architecture is sound.

``` text
opencode-observatory/
  apps/
    web/                    # Next.js UI and local API routes
  packages/
    domain/                 # Canonical domain types and validation
    source-adapters/        # OpenCode schema/API adapters
    ingestion/              # Incremental read-only sync
    analytics/              # Metrics, timelines and cost calculations
    pricing/                # Versioned pricing catalogue
    intelligence/           # AI providers, summaries and evidence links
    ui/                     # Shared components and design tokens
  data/                     # Local-only runtime data; gitignored
  docs/
    architecture.md
    security.md
    pricing-methodology.md
    local-installation.md
  tests/
```

## 12. Testing and acceptance criteria

### Source database integrity

-   Verify source database hash/schema and relevant record counts
    before/after tests where feasible.
-   Confirm read-only access prevents writes.
-   Test with OpenCode running and with a busy/WAL database.
-   Test legacy, V2 and overlapping/migrated data fixtures.

### Data accuracy

-   Reconcile session/message/token counts against OpenCode's own
    statistics where definitions match.
-   Prove idempotent sync: running it twice creates no duplicate records
    or costs.
-   Prove changed sessions update derived data and stale summaries are
    invalidated.
-   Validate parent/subagent relationships without double-counting.

### Cost accuracy

-   Unit-test token categories, rate-card versions, cache pricing,
    missing fields and currency handling.
-   Confirm API-equivalent values never appear as actual billed spend.
-   Verify free/included usage is shown separately from actual charges.
-   Show calculation provenance for every estimate.
-   Never default missing usage to zero.

### AI accuracy and safety

-   Validate output against a schema.
-   Evidence links must resolve to real source records.
-   Include uncertain/unverified outcomes.
-   Test transcript prompt injection and secret redaction.
-   Ensure remote AI processing cannot run without explicit
    configuration and user action.

### UX and reliability

-   Test filters, drill-down, global search, keyboard access, error
    states and responsive layouts.
-   Test restart/resume, interrupted import, corrupted dashboard cache
    and full rebuild.
-   Confirm no cloud dependency for local operation.
-   Confirm local server does not listen on a public interface by
    default.

### Definition of done

A release is acceptable only when: - the source database remains
untouched; - historical ingestion is reproducible and deduplicated; -
cost measures are clearly separated and auditable; - the four main
dashboard experiences are complete; - AI insights are evidence-backed
and optional; - automated tests pass; - Windows installation and
recovery documentation is complete; - known limitations are documented
honestly.

## 13. Delivery phases

1.  **Audit:** inspect repositories, licences, source schema and actual
    data; produce architecture decision record.
2.  **Foundation:** local app shell, source adapters, separate database,
    read-only incremental ingestion.
3.  **Core analytics:** overview, project/session explorer, activity
    timeline and model/token metrics.
4.  **Cost accounting:** actual/reported/estimated/hypothetical ledgers
    and free-model counterfactual pricing.
5.  **Intelligence:** opt-in summaries, outcomes, cross-session
    synthesis and evidence references.
6.  **Reliability:** incremental monitoring, diagnostics, recovery,
    search and performance.
7.  **Release:** security review, acceptance tests, Windows setup and
    future-hosting design notes.

Each phase must have a written summary, tests and known issues. Do not
start the next phase until the orchestrator reports completion and the
user approves it.

## 14. Open questions to resolve during Phase 1

Do not block the initial audit on these; record findings and propose
safe defaults: - Exact OpenCode version and actual schema present. -
Which providers/models are used and what billing evidence is
available. - Whether pricing should default to current rates or
historical rates. - Whether local models need user-defined comparable
hosted models. - Whether AI summaries should default to a local model or
remain disabled until configured. - Which data, if any, the user may
later choose to sync to a hosted version. - Which existing repository,
if any, is safe and efficient to reuse.

## 15. Final product standard

Build a real, usable application---not a static mockup. Do not replace
working code with placeholders, fabricate usage data, silently skip
unsupported schemas or claim completion without evidence. Prefer the
smallest well-tested implementation that satisfies the requirements, and
document every approximation.
