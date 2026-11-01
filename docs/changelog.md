# Changelog

All notable changes to OpenCode Observatory are recorded here. The version line
lives in [`VERSION`](VERSION).

## 1.0.0 — 2026-10-31

### Added

- Phase 1 audit deliverables: `phase-1-audit.md`, `architecture-decision-record.md`,
  `requirements-traceability.md`.
- Requirements traceability map covering every PRD and Master Prompt clause with a
  stable `PRD-<GROUP>-<NNN>` ID, target phase, verification method and honest status.
- Repository baseline for the public repository `collaby-org/opencode-observatory`:
  product README, MIT `LICENSE`, living-docs baseline and a `docs/VERSION` line at
  `1.0.0`.
- Audit evidence for the local OpenCode 2.2.4 source database (read-only inspection
  of a 2.33 GB WAL database with a migrated V2 layout) and for the three candidate
  open-source projects named in the Master Prompt.

### Changed

- `AGENTS.md` synchronized with the global agent workflow and `CLAUDE.md` reduced to
  the required `@AGENTS.md` pointer, so the repository standards check passes.
- Removed the starter-kit `UI_UX_MASTER.md` and `UI_UX_PROMPTS.md`, which described a
  generic redesign workflow rather than this product.
- Phase 1 deliverables corrected after the mandatory local review: the roadmap now
  names the Phase 2 minimal diagnostics shell and the Phase 6 incremental monitoring
  of new sessions, the provider and model open question records every identified
  provider, model id and provider/model pair from the read-only aggregate audit, the
  system diagram attaches the GET allowlist to the OpenCode HTTP API instead of the
  local API layer, ADR-010 no longer claims an unperformed shadcn/ui compatibility
  check, and the product promise keeps the free/included counterfactual separate
  from the other cost values.

### Notes

- No product feature is implemented. Phase 1 is audit and architecture only.
- The repository version starts at `1.0.0` because this is the first published
  baseline of a new repository. From the next change on, fixes and documentation
  bump the patch line and features or architecture bump the minor line.
