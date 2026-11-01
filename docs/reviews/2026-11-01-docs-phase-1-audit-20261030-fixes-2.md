# Code review: docs/phase-1-audit-20261030, 2026-11-01

**Verdict**: changes-requested
**Reviewed by**: nvidia/minimaxai/minimax-m3.1-flash (author model: nvidia/minimaxai/minimax-m3.1-flash)
**Scope**: 10 files, uncommitted against `docs/phase-1-audit-20261030`, head `ea13f6a6`
**Level**: standard · confidence gate 80 · passes: correctness, security, conventions

## Summary

Found 0 blocker(s), 1 major, 0 minor, 0 nit across 10 changed file(s). Highest severity: major in `docs/phase-1-audit.md:421`. The gate kept 1 finding(s) and dropped 0 (0 below confidence 80, 0 outside the changed lines).

## Major

### 🟠 tokenharbor model id is transcribed with a hyphen instead of a colon, leaving the table with 23 distinct model ids

`docs/phase-1-audit.md:421` · category: correctness · confidence: 100

**Problem**: The new provider/model identity table records tokenharbor's model as `deepseek-v4.1-flash-free`, but the aggregate source query output it was transcribed from records `tokenharbor|deepseek-v4.1-flash:free|5` (colon before `free`); the source's only tokenharbor DeepSeek v4.1 entry uses the colon form, while `deepseek-v4.1-flash-free` belongs to the opencode and openai pairs.

**Why it matters**: This is the exact identity evidence that closes PRD-OPEN-002 and feeds the Phase 4 provider/model-to-pricing mapping, and it records a provider/model pair that does not exist in the source while losing a distinct model id. It also makes the document contradict itself: the table now contains only 23 distinct model ids, while docs/phase-1-audit.md:402, the section 8 open-question row at line 520, the exit report at lines 691-692, docs/requirements-traceability.md:457 and docs/changelog.md:30 all state 24 distinct model ids, so a reader cannot reproduce the stated total from the published table.

**Suggested fix**: Change the row to `| `tokenharbor` | `deepseek-v4.1-flash:free` | 5 |` so the table matches the source and lists 24 distinct model ids; no other count changes.

**Evidence**: Added table row docs/phase-1-audit.md:421 reads `| `tokenharbor` | `deepseek-v4.1-flash-free` | 5 |`; source row in the aggregate query output is `tokenharbor|deepseek-v4.1-flash:free|5` (character 58 is ':', not '-'). A set comparison of all 30 table rows against the source rows reports this as the only difference; the other 29 rows, the 6 providers, the 30 pairs and the session total 1256 match exactly, and the distinct model ids counted from the table are 23 against the 24 claimed at docs/phase-1-audit.md:402, 520 and 691-692, docs/requirements-traceability.md:457 and docs/changelog.md:30.

## Coverage

- Files reviewed: 10
- Lines changed: +272 / -6
- Skipped files: 0 (lockfiles, generated output, binaries, snapshots)
- Chunks: 1
