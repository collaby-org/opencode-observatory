# Roadmap

The build follows the seven phases of the PRD. Each phase ends with a written
summary, tests and known issues, and needs explicit approval before the next one
starts.

## Phase 1 — Audit and architecture decision

- [x] Repository and environment audit, user changes preserved
- [x] Open-source evaluation of the three candidate repositories
- [x] Read-only audit of the local OpenCode source database
- [x] `docs/phase-1-audit.md`
- [x] `docs/architecture-decision-record.md`
- [x] `docs/requirements-traceability.md`
- [ ] Owner approval to start Phase 2

## Phase 2 — Data foundation

- [ ] Replace the starter-kit CI, Dependabot, CodeQL and secret-scan workflows with
  stack-aware ones and pin the secret scanner release
- [ ] Rewrite the issue templates, `CODEOWNERS` and `SECURITY.md` contact for this
  project; add `data/` and SQLite artefacts to `.gitignore`
- [ ] Create the TypeScript workspace: `apps/web` plus the domain, source-adapter,
  ingestion, analytics, pricing, intelligence and UI packages
- [ ] Implement the read-only source adapters for the audited V2 and legacy layouts,
  with explicit diagnostics for unsupported schemas
- [ ] Create the dashboard-owned SQLite database with raw and derived tables
  separated, migration metadata and indexes
- [ ] Implement incremental ingestion with checkpoints, idempotent upserts, resume and
  full rebuild
- [ ] Implement the deduplication rules for migrated legacy/V2 overlap, parent and
  child sessions, and forked sessions
- [ ] Build synthetic legacy-only, V2-only, migrated-overlap, parent/child, fork,
  missing-usage, partial-token and interrupted-sync fixtures and prove the
  source database is untouched before and after a sync
- [ ] Ship a minimal local diagnostics shell over the real source: detected
  source path and layout, import progress and row counts, warnings, and
  unavailable-usage counts

## Phase 3 — Core analytics

- [ ] Premium dark application shell, navigation, global filters and drill-down
- [ ] Overview with provenance-aware metrics and charts
- [ ] Project and session explorer with transcript viewer
- [ ] Agent activity timeline separating human prompts from recorded activity
- [ ] Model and token metrics

## Phase 4 — Cost accounting

- [ ] Six separate cost ledgers with per-amount provenance
- [ ] Versioned pricing catalogue with effective dates and source URLs
- [ ] Historical-rate and current-rate views
- [ ] Free and included usage counterfactual

## Phase 5 — Intelligence

- [ ] Optional, opt-in AI provider adapters with budgets and redaction
- [ ] Typed session and project summaries with evidence references
- [ ] Outcome status, confidence and regeneration

## Phase 6 — Reliability and operations

- [ ] Incremental monitoring of new sessions through a low-overhead, checkpointed
  watcher that reuses the ingestion path
- [ ] Reliability views, error trends and repeated-attempt heuristics
- [ ] Search, export, diagnostics, reset and recovery controls
- [ ] Performance work on a real dataset

## Phase 7 — Release

- [ ] Security review and acceptance suite
- [ ] Windows installation, backup and recovery guide
- [ ] Hosted-readiness assessment; no deployment without explicit authorisation
