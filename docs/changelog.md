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

### Notes

- No product feature is implemented. Phase 1 is audit and architecture only.
- The repository version starts at `1.0.0` because this is the first published
  baseline of a new repository. From the next change on, fixes and documentation
  bump the patch line and features or architecture bump the minor line.
