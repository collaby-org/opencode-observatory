# OpenCode Observatory

Local-first analytics and project-intelligence dashboard for
[OpenCode](https://opencode.ai). It turns the complete local session history
of OpenCode into a trustworthy dashboard: usage, tokens, cost provenance,
agent activity, outcomes and project intelligence.

> **Status: Phase 1 (audit and architecture decision).**
> No dashboard feature has been built yet. The product is built in seven gated
> phases and each phase needs explicit approval before the next one starts.
> Read [`docs/phase-1-audit.md`](docs/phase-1-audit.md) for the current state.

## What is in this repository right now

| Path | Purpose |
|---|---|
| [`PRD.md`](PRD.md) | Product requirements. This is the contract the build follows. |
| [`Master_Prompt.md`](Master_Prompt.md) | Orchestration contract: phases, gates, safety rules. |
| [`docs/phase-1-audit.md`](docs/phase-1-audit.md) | Repository, open-source and local source-data audit. |
| [`docs/architecture-decision-record.md`](docs/architecture-decision-record.md) | Accepted architecture decisions and rejected alternatives. |
| [`docs/requirements-traceability.md`](docs/requirements-traceability.md) | Every PRD clause mapped to a stable ID, phase and verification method. |
| [`docs/roadmap.md`](docs/roadmap.md) | Sequenced phases and their current status. |
| [`docs/tech_stack.md`](docs/tech_stack.md) | Runtimes and tools actually present. |
| [`docs/targets.md`](docs/targets.md) | Acceptance criteria and their status. |
| [`docs/project-doc.md`](docs/project-doc.md) | Product decisions and the reasoning behind them. |
| [`docs/system_diagram.md`](docs/system_diagram.md) | Intended component and data flow. |
| [`docs/changelog.md`](docs/changelog.md) | Dated change log. |

## Core promises

1. **The OpenCode database is never modified.** Observatory reads it through a
   read-only SQLite connection and writes everything derived into its own
   dashboard database.
2. **Every number has a provenance.** Actual billed spend, provider-reported
   usage, OpenCode-recorded cost, API-equivalent estimates and unknown values
   are separate ledgers and are never summed together.
3. **Local first.** The product runs on the local Windows machine, needs no
   cloud account, has no telemetry and uploads nothing by default.
4. **AI is optional.** Intelligence features stay off until a model is
   configured and the user asks for a summary.

## Requirements for the future local build

Not installed by this repository yet. Phase 2 records the exact pinned
versions in [`docs/tech_stack.md`](docs/tech_stack.md).

- Windows 10 or newer
- Node.js 24 LTS (the version verified during the Phase 1 audit)
- npm 11 (the only package manager working on the audited machine)

## Development workflow

The repository carries the `collaby-org/starter-kit` agent workflow. Every
change happens in its own worktree and branch, is reviewed locally, and lands
through a pull request.

```text
/scope -> /architect -> /develop -> /test -> /check verify -> review -> PR
```

See [`AGENTS.md`](AGENTS.md) for the full workflow and
[`COMMANDS.md`](COMMANDS.md) for the command index.

## License

MIT. See [`LICENSE`](LICENSE). The copyright holder line is still to be
confirmed by the repository owner. Third-party attribution obligations will be
recorded here if any external code is reused; the Phase 1 audit currently
recommends a clean-room implementation with no copied candidate code.
