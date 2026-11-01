# Phase 1 audit

**Project:** OpenCode Observatory
**Audit window:** 2026-10-30 to 2026-10-31
**Decision state:** Phase 1 complete; awaiting explicit owner approval
**Implementation state:** Phase 2 has not started

## 1. Purpose and scope

This report is the Phase 1 evidence record required by `Master_Prompt.md` and
`PRD.md`. It answers four questions before product code is written:

1. What repository and local environment will the product actually build on?
2. Can one of the three nominated open-source projects provide a safe and
   efficient starting point?
3. What schema and data does the local OpenCode installation actually expose?
4. Which architecture can satisfy the product contract without weakening the
   read-only source, provenance, privacy or phase-gate rules?

This phase deliberately makes **no feature implementation**. The only product
artefacts are the three Phase 1 documents:

- `docs/phase-1-audit.md` — this audit;
- `docs/architecture-decision-record.md` — the accepted and proposed decisions;
- `docs/requirements-traceability.md` — stable requirement identifiers and
  honest status.

The audit also established the repository baseline needed to make those
documents durable: a product-specific README, MIT licence, version line,
living documentation and a scope manifest. It did not scaffold an application,
install dependencies, open a product port, deploy anything or modify the
OpenCode source database.

### Safety boundary observed throughout

- The OpenCode database was opened only through a read-only SQLite URI and
  `PRAGMA query_only=ON`.
- No DDL or DML statement was issued against the source connection.
- Aggregate schema facts and JSON key names were inspected. Full transcripts,
  session titles, directory paths and credential values were not printed.
- Write refusal was proved against a scratch database, never the source.
- No candidate project was cloned, installed or executed in the application.
- No source data was uploaded, no product port was exposed and no deployment
  was attempted.

## 2. Method and evidence

### Method

The audit used four independent evidence streams:

1. **Repository inspection** — git state, tracked files, workflow configuration,
   repository standards and the actual runtime/toolchain.
2. **Candidate repository review** — GitHub metadata, licence files, dependency
   manifests, selected source files and CI/test inventories through the GitHub
   API. No candidate was installed.
3. **Read-only source inspection** — reproducible Python audit scripts that
   report schema metadata, aggregate counts and payload key names only.
4. **Architecture review** — an independent read-only review compared the
   candidate architectures with the PRD, the verified environment and the
   observed schema.

### Evidence hierarchy

| Claim | Primary evidence | Confidence |
|---|---|---|
| The baseline has no application | Tracked-file inventory and absence of a package manifest or lockfile | High |
| Candidate licences and current activity | GitHub repository and licence API responses on 2026-10-31 | High |
| Candidate implementation behaviour | Direct review of relevant source files and manifests | High for inspected paths |
| Installed OpenCode version | Desktop CLI version and authenticated `GET /api/info` | High |
| Installed API shape | Authenticated `GET /openapi.json` | High |
| Source schema and counts | Two read-only aggregate audit scripts | High for the recorded snapshot |
| Payload field names | JSON key-name inspection of sampled rows | High for the sampled shapes; not a guarantee of every historical row |
| Read-only refusal | Write attempted against a scratch database using the same read-only mechanism | High |
| Future OpenCode schema | Current official V2 documentation and upstream source | Medium over time; every adapter must detect at runtime |

### Exact commands and checks used

The following commands were run during the audit or its final verification. The
service credential was read from the local service registration only for the
authenticated request and was never printed.

```powershell
git status --short
git branch --show-current
git log -1 --oneline
git remote -v
gh pr list --state open --json number,title,url,headRefName

git --version
node --version
npm --version
python --version
dotnet --version
go version
sqlite3 --version
pnpm --version
yarn --version
bun --version
Get-Command opencode -ErrorAction SilentlyContinue

node .agents/skills/repo-standards/scripts/standards.mjs --check .
node "$env:USERPROFILE\.config\opencode\tools\agents-scope.mjs" --check .agents/lanes.json

python "$env:TEMP\opencode\source_db_audit.py" > "$env:TEMP\opencode\after1.json"
python "$env:TEMP\opencode\source_db_audit2.py" > "$env:TEMP\opencode\after2.json"

gh api repos/dwrdx/opencode-session-inspector
gh api repos/Amanuel-Tesfaye-R/opencode-usage-tracking
gh api repos/Andres77872/opencode-dashboard
gh api repos/<owner>/<repo>/license
gh api repos/<owner>/<repo>/contents/<path>

$svc = Get-Content "$env:USERPROFILE\.local\state\opencode\service.json" -Raw | ConvertFrom-Json
$pair = "opencode:$($svc.password)"
$b64 = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($pair))
$headers = @{ Authorization = "Basic $b64" }
Invoke-RestMethod -Uri "$($svc.url)/api/info" -Headers $headers
Invoke-RestMethod -Uri "$($svc.url)/openapi.json" -Headers $headers
curl.exe -s -o NUL -w "%{http_code}" -H "Authorization: Basic $b64" `
  "$($svc.url)/api/experimental/migration/v1/status"
```

The two Python scripts are outside the repository because Phase 1 publishes
findings, not an application or a reusable product package. Their source is
retained in the approved temporary working directory so the audit is
repeatable. Phase 2 will turn the relevant behaviours into repository-owned,
synthetic fixture tests.

### Limits of this evidence

- The source database is live. Its row counts and WAL contents can change
  between readings. Counts below are therefore point-in-time observations, not
  immutable product facts.
- A full-file hash of a multi-gigabyte WAL database while OpenCode is running
  is not a stable integrity signal. The audit used schema, counts, file
  metadata and a first-megabyte fingerprint. A stopped-source hash comparison
  is deferred to Phase 2.
- The candidate review inspected relevant implementation, not every historical
  commit. Current licence and maintenance signals were rechecked on the audit
  date.
- No application existed, so there were no application lint, typecheck, unit,
  integration, build or browser checks to run. Saying otherwise would fabricate
  evidence.

## 3. Repository and environment audit

### Repository state

| Item | Finding |
|---|---|
| Public repository | `collaby-org/opencode-observatory` |
| Default branch | `main` |
| Baseline commit | `6428a6e` from the starter-kit baseline |
| Phase 1 branch | `docs/phase-1-audit-20261030` |
| Open pull requests | None at final check |
| Application source | None |
| Package manifest or lockfile | None |
| Test, lint, typecheck or build commands | None |
| Tracked files | 2,205, of which 2,171 are under `.agents/` |

The inherited repository is a workflow and standards kit, not an application.
Its generic README describes the starter-template, secrets and UI/UX-redesign
workflow. Its issue templates describe another product, `CODEOWNERS` and the
security contact contain placeholders, and CI, Dependabot and CodeQL target
absent PHP, Laravel, frontend and gateway application directories. The
secret-scan workflow installs an unpinned scanner release.

The user supplied `PRD.md` and `Master_Prompt.md`; both were preserved as the
product and orchestration contracts. Two obsolete UI-template documents were
already removed by the user in the source checkout and that deletion was kept.
No user change was discarded.

The repository standards check initially found two concrete issues: the global
and repository `AGENTS.md` files differed, and `CLAUDE.md` did not contain
exactly the required pointer. The owner explicitly authorised genericising the
machine-specific section of the global instructions and synchronising them.
After that authorised change, the check returned:

```text
OK: all 13 required files present and valid
```

### Host and tools

| Tool | Result | Consequence |
|---|---|---|
| Windows | Available | Primary local target |
| Git | 2.55.0.windows.5 | Worktree and PR workflow available |
| Node.js | v24.21.0 | Supports the proposed TypeScript runtime |
| npm | 11.19.0 | The only working package-manager path |
| Python | 3.14.7 | Used for the reproducible read-only audit scripts |
| .NET SDK | 10.0.401 | Present but not selected for the product |
| Go | Absent | Candidate 3 cannot be built or tested here without a new toolchain |
| SQLite CLI | Absent | Verification used Python's SQLite module |
| pnpm and Yarn | Broken global shims | Neither is a viable project baseline |
| Bun | Absent | Not selected |
| `opencode` on `PATH` | Absent | The effective CLI must be discovered, not assumed |

### OpenCode installation

The effective installation is OpenCode **2.2.4**, exposed by the desktop
application's bundled CLI. A separate older 1.14.22 CLI exists, has no V2
service command and is not on `PATH`. The running service is registered at a
loopback URL and uses Basic authentication. The credential was not recorded in
this report.

This split installation is operationally important: a product that shells out
to `opencode` could select the wrong binary or fail because it is not on
`PATH`. The Phase 2 source-discovery work must therefore cover the documented
database location independently of CLI discovery.

## 4. Template suitability

The starter-kit is **useful as workflow infrastructure but unsuitable as an
application baseline**. The decision is to retain and adapt its engineering
workflow while replacing application-specific assumptions as each phase needs
them.

### Retained

- Worktree, branch, scope-manifest and pull-request discipline.
- Repository standards, issue forms, ownership and secret-scanning concepts.
- Agent workflows, provided they are re-evaluated for the actual TypeScript
  stack and never treated as product test evidence by themselves.
- The living documentation pattern now represented by `docs/`.

### Replaced or deferred

| Template element | Finding | Action |
|---|---|---|
| Generic project README | Describes an unrelated product | Replaced with an Observatory-specific baseline |
| No root licence | README claimed MIT without a licence file | Added MIT `LICENSE` |
| PHP/Laravel/frontend/gateway CI | Targets absent directories | Replace with stack-aware CI in Phase 2 |
| Dependabot directories | Targets absent application roots | Replace when the workspace exists |
| CodeQL PHP mode | Wrong language | Replace when application source exists |
| Unpinned secret scanner | Supply-chain weakness | Pin release and checksum in Phase 2 |
| Unrelated issue templates and placeholders | Wrong owner/product context | Rewrite in Phase 2 |
| Empty documentation roots | Useful locations only | Populate only with real, verified artefacts |
| Legacy `opencode.json` shape | Deprecated keys | Migrate deliberately when product configuration is needed |

The adopted architecture is consequently a clean TypeScript workspace rather
than a Laravel-plus-gateway system. This is an evidence-based deviation from
the inherited template, not an unrelated refactor.

## 5. Open-source evaluation

All three candidates were reviewed through their current GitHub state and
relevant source files. None was cloned, installed, built or executed.

### 5.1 `dwrdx/opencode-session-inspector`

| Attribute | Finding |
|---|---|
| Licence | MIT, copyright notice present |
| Current signal | 1 star, 0 forks, 9 commits, last push 2026-09-23, no releases or CI history |
| Stack | TypeScript, Node 24, `node:sqlite`, no runtime dependencies, vanilla browser UI |
| Schema | Runtime table detection; V2-first with legacy fallback |
| Tests | None |
| Security posture | Loopback default and bounded request body are positive |

**Disposition: rejected as a base.** It is a session manager rather than an
analytics product and, critically, opens the source database without
read-only mode and deletes sessions from it. Reusing its database layer would
violate the most important product invariant. Its runtime schema detection and
V2-first fallback are useful ideas that can be independently reimplemented.

### 5.2 `Amanuel-Tesfaye-R/opencode-usage-tracking`

| Attribute | Finding |
|---|---|
| Licence | MIT, copyright notice present |
| Current signal | 1 star, 0 forks, 27 commits, last push 2026-08-17, no releases, CI or tests |
| Stack | Next.js 16, React 19, strict TypeScript, Tailwind v4, Recharts, `node:sqlite` |
| Schema | Legacy `session`, `message` and `part` tables only; V2 unsupported |
| Persistence | Queries the live source on every request; no dashboard database, checkpoint or upsert |
| Cost | Sums stored cost and renders it as dollars; token totals are internally inconsistent |

**Disposition: rejected as a base.** It misses the current V2 schema, lacks a
separate store and durable ingestion, and starts its Next.js server on all
interfaces without authentication while exposing prompts, paths and tool
output. Its query approach would both miss new sessions and create an unsafe
network posture. Visual and metric-decomposition ideas are the only useful
parts.

### 5.3 `Andres77872/opencode-dashboard`

| Attribute | Finding |
|---|---|
| Licence | Apache-2.0, notice and modification obligations apply to copied material |
| Current signal | 2 stars, 1 fork, at least 100 commits, 52 releases, 53 tags, 59 Actions runs |
| Stack | Go 1.26 backend, React/Vite frontend, `modernc.org/sqlite`, TUI, five non-OpenCode sources |
| Source safety | Read-only DSN, query-only pragma, WAL and busy-timeout handling |
| Cost | Per-source provenance and no combined misleading spend total |
| Tests | 142 Go test files, 29 frontend test files and synthetic source fixtures |
| Release gap | Linux and macOS only; no Windows binary |

**Disposition: adapt for architecture, do not reuse as the base.** It is the
strongest reference and validates the read-only adapter, layout detection,
fork handling, missing-usage semantics, per-ledger provenance, fixtures and
host-header protection. It cannot be adopted wholesale because Go is absent,
Windows is unsupported by its releases, its scope is much wider, and its
analytics-agent behaviour does not match the PRD's typed evidence model.

### Reuse conclusion

No candidate code, assets, SQL or generated files will be copied in the
initial build. Ideas are not code, so the clean-room implementation can retain
the proven architectural patterns without importing the candidates' licence
obligations, dependency trees or unsafe behaviour. If that decision changes,
the applicable MIT notice or Apache-2.0 licence, notice and modified-file
marking must be handled before any code enters the repository.

## 6. Local source data audit

### Location and safety proof

The source database is at the documented default location
`%USERPROFILE%\.local\share\opencode\opencode.db`; `OPENCODE_DB` and
`XDG_DATA_HOME` were unset. At the initial audit it was 2,334,752,768 bytes,
used WAL journalling, and had `user_version=0` and `application_id=0`. A live
WAL and shared-memory file were present, confirming that OpenCode was writing
while the audit read.

The audit connection used:

```text
file:<source path>?mode=ro
PRAGMA query_only = ON
PRAGMA busy_timeout = 5000
```

Observed values were `query_only=1` and `journal_mode=wal`. Concurrent reads
succeeded. A write against a scratch database opened by the same mechanism was
refused with `attempt to write a readonly database`. No write statement was
issued against the source.

### Snapshot schema and row counts

There are 30 application tables. The following counts are from the initial
2026-10-30 snapshot:

| Table | Rows | Table | Rows |
|---|---:|---|---:|
| `session_v2` | 1,250 | `session_message` | 111,797 |
| `session_part` | 0 | `session` | 148 |
| `message` | 13,433 | `part` | 59,297 |
| `project` | 54 | `project_directory` | 8 |
| `worktree` | 41 | `workspace` | 0 |
| `session_inbox` | 37 | `todo` | 123 |
| `permission` | 104 | `event` | 853 |
| `event_sequence` | 1,265 | `kv` | 600 |
| `migration` | 59 | `__drizzle_migrations` | 13 |
| `credential` | 9 | `blob` | 0 |

The database is live, so these numbers naturally drift. A later same-window
verification observed 1,254 V2 sessions, 112,249 V2 messages and 1,089
parent-linked V2 sessions, while the legacy counts and the 140-session overlap
were unchanged. That growth is a useful reminder that every dashboard metric
must be a snapshot with a sync time, not a timeless property of the source.

### Layout and migration state

`kv['migration.v1-v2']` records phase `completed`. The effective source is
therefore **V2 plus a retained legacy snapshot**:

- 140 of 148 legacy session IDs also exist in V2, and their legacy message and
  part rows are duplicated there.
- Eight legacy-only sessions remain and must not be dropped.
- Reading both layouts without reconciliation would double count the 140
  overlapping sessions.
- `session_part` is empty. In V2, transcript payload is embedded in
  `session_message.data`.

The chosen rule is V2-authoritative for the overlapping 140 sessions, with
the eight legacy-only sessions retained. Source IDs and layout provenance
remain attached to every canonical record.

### Relationships and worktree evidence

At the initial snapshot, `parent_id` was populated on 1,085 of 1,250 V2
sessions across 35 distinct parents. Parent and child sessions remain
individually traceable, but usage must be attributed to the request that owns
it rather than summed from both session aggregates.

`workspace_id` was null on every V2 session. Worktree relationships therefore
cannot be inferred from that column; the adapter must use the `worktree` table
and session directory together and leave a relationship unknown when the
evidence is insufficient.

`fork_session_id` and `fork_boundary` were null everywhere. Fork-copy
exclusion is required by the architecture and must be fixture-tested, but it
is **unverified against current local source data**. This is recorded as an
unknown, not as proof that fork behaviour is absent.

### Usage, cost and model evidence

- `session_v2.cost` and `tokens_input` were populated on all initial 1,250
  sessions.
- 1,218 sessions had model JSON, representing 45 distinct model values.
- A later snapshot resolved those payloads into 6 providers, 30 provider/model
  pairs and 24 distinct model ids across 1,256 sessions with a model payload.
  The raw payload strings carry provider-specific variants, so the raw count
  above is higher than the pair count. Only provider and model identities were
  extracted; no transcript, title or path data is involved.

  | Provider | Model id | Sessions |
  |---|---|---:|
  | `opencode` | `beam-free` | 799 |
  | `opencode` | `deepseek-v4.1-flash-free` | 100 |
  | `nvidia` | `stepfun-ai/step-5-preview` | 99 |
  | `opencode` | `glm-5.3-flash-free` | 70 |
  | `nvidia` | `minimaxai/minimax-m3.1-flash` | 43 |
  | `opencode` | `mimo-v2.6-flash-free` | 30 |
  | `tokenharbor` | `mimo-v2.5:free` | 22 |
  | `openrouter` | `minimax/minimax-m3.1-flash:free` | 14 |
  | `nvidia` | `z-ai/glm-5.3-flash` | 12 |
  | `nvidia` | `deepseek-ai/deepseek-v4.1-flash` | 11 |
  | `opencode` | `nemotron-3.5-super-vl-free` | 8 |
  | `openrouter` | `openrouter/free` | 8 |
  | `tokenharbor` | `deepseek-v4.1-flash:free` | 5 |
  | `nvidia` | `nvidia/nemotron-3.5-super-vl-120b-a12b` | 4 |
  | `openai` | `beam-free` | 4 |
  | `opencode` | `big-pickle` | 4 |
  | `openai` | `deepseek-v4.1-flash-free` | 3 |
  | `openai` | `mimo-v2.6-flash-free` | 3 |
  | `opencode` | `nemotron-3.5-lightning-free` | 3 |
  | `nvidia` | `nvidia/nemotron-3-ultra-550b-a55b` | 2 |
  | `opencode` | `flint-40b-free` | 2 |
  | `openai` | `glm-5.3-flash-free` | 2 |
  | `oc-zen` | `beam-free` | 1 |
  | `oc-zen` | `minimax-m3.1-flash-free` | 1 |
  | `opencode` | `claude-opus-5-5` | 1 |
  | `opencode` | `claude-sonnet-5-5` | 1 |
  | `opencode` | `gpt-6-luna` | 1 |
  | `opencode` | `muse-spark-1.3` | 1 |
  | `tokenharbor` | `muse-spark-1.3` | 1 |
  | `tokenharbor` | `deepseek-v4-flash:free` | 1 |

  Session counts drift with the live source; the identities and the mapping are
  the durable finding, and Phase 2 re-discovers them at runtime.
- In the 5,000-row assistant sample captured during the Phase 1 audit, 4,933
  rows had a cost and token object and 59 carried an error.
- V2 assistant payload keys include `agent`, `content`, `cost`, `error`,
  `finish`, `model`, `rawFinish`, `retry`, `snapshot`, `time` and `tokens`.
- The initial `session_message.type` distribution was assistant 101,939, user
  2,976, idle 2,880, synthetic 2,368, system 1,391, model-switched 116,
  compaction 108, location-switched 15 and agent-switched 4.
- The sampled V2 `user` payload keys were `text`, `files` and `time.created`;
  the sampled `idle` payload keys were `outcome` and `time.created`. Runtime
  inspection must re-confirm these row contracts before Phase 2 implements
  them.
- Token categories include input, output, reasoning, cache read and cache
  write.
- Compaction rows are model requests with their own cost and token evidence.
- Legacy `part.data` carries tool state, commands, file paths, diffs, errors
  and timings. That content is evidence to display safely, never an
  instruction to execute.

The 59 sampled errors are the decisive missing-usage case: such requests must
be represented as **usage unavailable**, never as zero or free.

### Model catalogue and sensitive tables

`kv['models-dev:v2:catalog']` held about 11.3 MB of models.dev data with a
separate metadata digest and ETag. It is a useful model-identity input and a
candidate pricing-catalogue input, but not an authoritative, effective-dated
rate card for Observatory.

The `credential` table contains authentication material. Observatory must never
select, read, transform, export or display it. Its existence is recorded only
to establish that the table is in the source schema and out of scope.

## 7. OpenCode runtime and API audit

### Version and service

| Check | Result |
|---|---|
| Effective desktop-bundle CLI | 2.2.4 |
| Other CLI found on disk | 1.14.22, without the V2 service command |
| Service registration | Present |
| Service address | Loopback only |
| Authentication | Basic authentication; credential redacted |
| `GET /api/info` version | 2.2.4 |

### Installed OpenAPI surface

The installed service returned OpenAPI **3.1.0** with **138 paths** and **362
schemas**. Verified read endpoints include `/api/info`, session listing and
detail, messages, context, diff, inbox, log, experimental session statistics
and export, and model listing.

The same API exposes mutating and control operations, including session
creation, deletion, patching, prompting, forking, compaction, reversion and
inbox deletion. Observatory must therefore use an explicit GET allowlist and
must never proxy the OpenCode API.

Current official V2 documentation described a slightly larger surface than the
installed 2.2.4 service. The optional migration-status endpoint documented
upstream returned **HTTP 404** on the installed build. Every optional endpoint
must be feature-detected, and its absence must not be treated as a product
failure.

### API use in the product

The local database is the primary historical source because it contains the
per-request usage payload needed for auditable cost breakdowns. The API can
supplement selected session or live information through the narrow read
allowlist. It is not a replacement for the database and is never exposed to
the browser as a general proxy.

## 8. Open questions

All seven Phase 1 questions in `PRD.md` section 14 now have safe defaults.

| Question | Resolution |
|---|---|
| Exact OpenCode version and schema | OpenCode 2.2.4; 30 tables; V2 plus completed legacy snapshot |
| Providers, models and billing evidence | 6 providers, 30 provider/model pairs and 24 distinct model ids identified in section 6; no authoritative invoice or billing source found locally |
| Current or historical pricing default | Current rates by default, with a selectable historical view that never overwrites the original calculation |
| Comparable hosted model for local models | Not applicable/unknown until the user explicitly chooses a hosted model and rate card |
| AI default | Disabled until a local or remote model is configured and the user requests processing |
| Data eligible for later hosted sync | None selected now; any future sync is explicit, opt-in, authenticated, encrypted, revocable and separately reviewed |
| Existing repository safe to reuse | No candidate adopted; clean-room TypeScript build, candidate 3 used only as an architectural reference |

The absence of authoritative billing evidence means the actual-billed and
provider-reported ledgers stay unknown. It does not justify converting a
stored OpenCode value into “actual spend”.

## 9. Risks and unknowns

| Risk or unknown | Impact | Mitigation and owner |
|---|---|---|
| Schema or payload drift | Wrong counts, missing evidence or silent omissions | Runtime introspection, explicit unsupported-schema diagnostics and fixture variants; source adapters owner in Phase 2 |
| Legacy/V2 double counting | Inflated session, token and cost totals | V2 precedence, legacy-only retention, stable ownership rules and reconciliation; ingestion owner |
| Parent/child overlap | Duplicated activity and cost | Attribute usage to the owning request; keep sessions traceable but do not sum parent and child aggregates; ingestion and analytics owners |
| Fork semantics unverified locally | Copied history may be counted | Fixture-test boundary exclusion, preserve fork provenance and quarantine ambiguity; ingestion owner |
| Accidental source mutation | Irreversible loss of the user's record | Read-only URI, query-only mode, parameterised SQL, refusal tests and diff review; every phase owner |
| Local API exposure | Transcript and path disclosure | Loopback binding, Host and Origin checks, no general API proxy; application owner |
| Windows SQLite compatibility | Foundation blocked or unstable | Early `node:sqlite` spike with a fallback decision before the data layer expands; platform owner |
| Live WAL contention | Lock errors or inconsistent snapshots | Batched consistent reads, bounded retries, busy timeout, checkpoint diagnostics and later measurement; ingestion owner |
| Cost-category confusion | False billing claim | Six separate ledgers, per-amount provenance, no combined spend total; cost owner in Phase 4 |
| Missing usage shown as zero | Understates usage and cost | Null or unavailable semantics at every boundary; analytics owner |
| Secret leakage in history | Privacy and account exposure | Redaction before display, export and AI processing; security owner from Phase 2 |
| Hostile transcript instructions | Remote execution or unsafe AI behaviour | Treat history as untrusted data, no execution, typed output, injection and redaction tests; intelligence and security owners |
| Template CI remains misleading | False green or broken checks | Replace application-specific workflows in Phase 2 and report only commands actually run; repository owner |
| Database growth | Slower imports and UI | Incremental processing, indexed queries, real-data benchmarks and pagination; performance owner in Phase 6 |

None of these risks blocks the Phase 2 foundation. The only genuine Phase 2
gating decision is the SQLite client spike, and ADR-012 defines that decision
gate before broad implementation.

### Blocked items

No requirement is Blocked by Phase 1. Every Phase 1 prerequisite named by the
traceability matrix has now been answered. The remaining items are ordinary
implementation work in their approved future phases; they are not blocked by
an unresolved audit question.

## 10. Baseline metric verification

“Baseline” here means the small set of claims Phase 2 must establish before
later metrics can be trusted. It does not mean fabricated demo data.

| Claim to establish | Minimum decisive evidence | Boundary cases | Owner |
|---|---|---|---|
| The source is never modified | Scratch write refusal; stopped fixture hash and schema comparison before and after import; live read-only check | WAL, busy and interrupted reads | Phase 2 implementation; orchestrator verifies |
| Layout detection is truthful | Runtime table/column detection against synthetic legacy, V2 and mixed fixtures | Missing table, changed column, unsupported payload | Source-adapter owner |
| Overlap is reconciled | One canonical session per overlapping source ID; eight-style legacy-only records retained with layout provenance | Overlap, legacy-only and ambiguous records | Ingestion owner |
| Parent/child usage is not doubled | Request-level ownership fixture reconciled with parent and child aggregates | Many children, retries and missing usage | Ingestion and analytics owners |
| Sync is idempotent | Two consecutive imports produce identical canonical and ledger counts and stable source fingerprints | Empty second run and partial prior state | Ingestion owner |
| Sync resumes | Interrupted fixture resumes from checkpoint without duplicates | Crash between batches and corrupt derived row | Ingestion owner |
| Missing usage stays missing | Null/unavailable assertion for errored assistant and compaction rows | Partial token categories and malformed payload | Analytics owner |
| Session totals reconcile | Request-derived totals match `session_v2` aggregates where definitions match; differences are documented | Compaction, retries and parent/child sessions | Analytics owner |
| The dashboard store is disposable | Delete and rebuild the Observatory database while the source fingerprint is unchanged | Empty store and interrupted rebuild | Data-store owner |
| Local exposure is constrained | Server binds loopback and rejects foreign Host and Origin headers | `localhost`, bound address and malicious Host | Application and security owners |
| The Windows build works | npm install, typecheck, unit, integration and production build all run on Windows | Clean checkout and Node 24 | Repository owner |

The verification path deliberately starts with fixtures. The live database is
used for compatibility and reconciliation, but it is not mutated and not used
to manufacture expected results. A missing evidence item remains a gap; it is
not converted into a claim.

## 11. Proposed Phase 2 scope

Phase 2 is the smallest coherent foundation that can prove the source-safety
and data-correctness claims before any polished product surface is built.

### In scope

1. **Repository foundation**
   - Replace application-specific CI, Dependabot, CodeQL and secret-scan
     assumptions with stack-aware configuration.
   - Rewrite issue templates, ownership and the security contact for this
     project.
   - Ignore `data/`, database, WAL and shared-memory artefacts.
2. **TypeScript workspace**
   - `apps/web` plus focused domain, source-adapter, ingestion, analytics,
     pricing, intelligence and UI packages under npm workspaces.
3. **Source discovery and diagnostics**
   - Resolve the effective database through documented paths and an explicit
     selector.
   - Detect schema, migration state and supported layout at runtime.
   - Report unsupported schemas rather than guessing.
4. **Read-only adapters**
   - Use `mode=ro`, query-only mode, busy timeout and bounded retries.
   - Emit canonical session metadata and model-request usage first; full
     transcript and tool-part ingestion follows only after those paths are
     stable.
   - Never select the `credential` table.
5. **Dashboard-owned SQLite store**
   - Own migrations and metadata.
   - Keep raw and derived records logically separated.
   - Add the indexes justified by actual query plans.
6. **Checkpointed ingestion**
   - Initial backfill with progress and cancellation.
   - Idempotent upserts, interruption and resume, changed-session detection,
     controlled reconciliation and full rebuild.
   - Apply the accepted V2-precedence, legacy-only, parent/child and future
     fork rules.
7. **Synthetic fixtures**
   - Legacy, V2, migrated overlap, legacy-only, parent/child, fork,
     missing-usage, partial-token, malformed-payload and busy-WAL fixtures.
8. **Minimal local shell**
   - Show selected source, detected layout, import progress, imported and
     updated counts, warnings and unavailable-usage counts.
   - No premium charts or complete explorer in this phase.

### Explicitly deferred

Premium dashboard pages, full transcript ingestion, the live monitoring
daemon, the complete pricing catalogue, AI summaries, exports, hosted mode and
multi-source support remain in their later approved phases.

### Phase 2 exit criteria

- Source write refusal and stopped-fixture integrity checks pass.
- Migrated fixtures produce one canonical record per overlap and retain
  legacy-only records with source and layout identifiers.
- Parent and child sessions remain traceable without duplicated usage.
- Two imports produce identical counts; an interrupted import resumes without
  duplication.
- Missing usage remains unavailable and never zero.
- Request-derived totals reconcile with session aggregates where definitions
  match, with documented differences.
- The dashboard database can be rebuilt without touching the source.
- Windows install, typecheck, tests and build succeed with npm workspaces.
- The local server binds loopback and rejects disallowed Host headers.

Phase 2 still requires explicit owner approval. **Phase 2 has not started.**

## 12. Phase 1 exit report

### 1. What was inspected

- The full repository state, tracked-file inventory, git branches and remotes,
  user changes, standards, workflow definitions, issue templates, ownership,
  security guidance, dependency automation and secret scanning.
- The Windows host, available runtimes and package managers, the effective and
  stale OpenCode CLIs, the registered service and its installed API.
- All three candidate repositories: current metadata, licences, manifests,
  selected implementation paths, schema handling, ingestion, cost behaviour,
  security posture, tests and release signals.
- The live OpenCode source database through two read-only aggregate scripts:
  30 tables, columns, indexes, row counts, migration state, layout overlap,
  relationships, usage payload key names, model identities and read-only
  refusal.

### 2. Evidence and exact commands used

The exact command set is listed in section 2. The most important results were:

- Repository standards: `OK: all 13 required files present and valid`.
- Scope manifest: seven lanes, three writers and four read-only lanes with no
  overlapping write scope.
- Source scripts: completed successfully; `query_only=1`, WAL mode, 30 tables,
  main-file size unchanged across the read window, and scratch write refused.
- Installed API: version 2.2.4; OpenAPI 3.1.0; 138 paths; 362 schemas;
  optional migration-status request returned HTTP 404.
- Candidate metadata: current GitHub API responses for all three repositories,
  including the MIT and Apache-2.0 licence identifiers.
- Open pull requests at final check: none.

### 3. What was found

- The inherited repository contains no application; only a workflow kit.
- Node 24 and npm work, while Go is absent and pnpm and Yarn are broken.
- The effective OpenCode version is 2.2.4, not the stale 1.14.22 binary.
- The source is a 2.33 GB WAL database with V2 and a completed legacy snapshot.
- The audited model payloads resolve to 6 providers, 30 provider/model pairs and
  24 distinct model ids, listed in section 6; no local billing source exists.
- 140 sessions overlap across layouts, eight are legacy-only, and parent links
  dominate the relationship graph.
- Error records exist without usage and must remain unavailable, not zero.
- The installed API mixes useful reads with dangerous mutating operations.
- Each candidate has a decisive incompatibility: source mutation, legacy-only
  schema with unsafe exposure, or no Windows build and excessive scope.

### 4. Decisions and rationale

- **Clean-room TypeScript build.** It matches the working environment and the
  PRD's engineering direction without adding Go or a second service boundary.
- **No candidate code copied.** The useful patterns can be reimplemented while
  avoiding unsafe code and unnecessary attribution obligations.
- **Read-only source plus separate store.** The source remains the user's
  record; Observatory owns every derived record.
- **V2 precedence with legacy-only retention.** This reconciles overlap
  without losing the eight historical sessions.
- **Six cost ledgers.** Stored cost is not actual billed spend, and estimates
  are never merged with measured values.
- **AI and hosted mode remain explicit future capabilities.** Neither is
  allowed to weaken local privacy or the approval gates.

### 5. Risks and unresolved questions

The principal risks are schema drift, deduplication mistakes, parent/child
overlap, unverified local fork semantics, live WAL contention, Windows SQLite
compatibility, accidental source mutation or exposure, cost confusion, secret
leakage and stale template automation. Each has an owner and a verification
route in sections 9 and 10.

Remaining implementation questions are deliberately bounded: the exact
`node:sqlite` client decision is an early Phase 2 spike, and all future
schema support is discovered at runtime. The absence of authoritative billing
evidence is not an unknown to guess around; it means those ledgers remain
empty/unknown until evidence exists.

### 6. Proposed Phase 2 scope

Build the Windows-local TypeScript foundation: source discovery and runtime
layout detection, read-only adapters, the separate dashboard database,
checkpointed idempotent ingestion and reconciliation, synthetic fixtures and
a minimal diagnostics shell. The exact inclusions, deferrals and exit criteria
are in section 11. No Phase 2 implementation begins without approval.

### 7. Checks performed and results

| Check | Result |
|---|---|
| Repository standards | Passed — all 13 required files valid |
| Lane scope check | Passed — no overlapping write scope |
| Read-only source audit, part 1 | Passed — `query_only=1`, WAL read succeeded, main-file size unchanged |
| Read-only source audit, part 2 | Passed — aggregate and key-name report completed; scratch write refused |
| OpenCode API inspection | Passed — 2.2.4, OpenAPI 3.1.0, 138 paths, 362 schemas |
| Optional migration endpoint | Feature gap confirmed — HTTP 404 on installed 2.2.4 |
| Candidate metadata and licence recheck | Passed for all three current repositories |
| Open pull-request duplicate check | Passed — none open |
| Application lint, typecheck, unit, integration, build and browser tests | Not run — no application source exists in Phase 1 |
| Kodus local review | Not run — local CLI installation is broken and no team key is configured |
| PR-Agent review | Deferred to the pull-request workflow; no clean result is claimed here |

**Phase 1 conclusion.** The audit is complete, the three required deliverables
are present, the architecture decisions are recorded, and no product feature
has been implemented. Work is stopped at the mandatory approval gate. The
owner must explicitly approve Phase 2 before the proposed foundation work
begins.
