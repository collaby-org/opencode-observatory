# Targets

Acceptance criteria for the product and their current status. The full
requirement-by-requirement mapping with verification methods lives in
[`requirements-traceability.md`](requirements-traceability.md).

Status markers: `done`, `open`, `blocked`.

## Phase 1, audit

- [x] done — Repository, environment and template audited without discarding user changes
- [x] done — Three candidate repositories evaluated for licence, maintenance, schema, cost, security and tests
- [x] done — Local OpenCode source database inspected read-only, with schema, counts and migration state recorded
- [x] done — Architecture decisions recorded in [`architecture-decision-record.md`](architecture-decision-record.md)
- [x] done — Requirements traceability published
- [ ] open — Owner approval to start Phase 2

## Release gates, from the PRD definition of done

- [ ] open — The source database remains untouched
- [ ] open — Historical ingestion is reproducible and deduplicated
- [ ] open — Cost measures are clearly separated and auditable
- [ ] open — The four main dashboard experiences are complete
- [ ] open — AI insights are evidence-backed and optional
- [ ] open — Automated tests pass
- [ ] open — Windows installation and recovery documentation is complete
- [ ] open — Known limitations are documented honestly

## Product promises that must hold at every phase

- [ ] open — The OpenCode database is never written, migrated, vacuumed or altered
- [ ] open — The local server binds loopback by default and rejects foreign Host headers
- [ ] open — Nothing is uploaded and no telemetry is sent by default
- [ ] open — Actual, provider-reported, recorded, estimated and unknown cost values are never merged
- [ ] open — Missing usage is reported as unavailable, never as zero
- [ ] open — AI processing stays off until a model is configured and the user asks for it
- [ ] open — No transcript content is ever executed as a command
