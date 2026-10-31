# Tech stack

Only what is actually in use today, plus the stack the Phase 1 audit accepted
for the build. Planned entries are marked as planned and are not installed.

## Verified in this environment

| Tool | Version | Evidence |
|---|---|---|
| Windows | 10 or newer | Host of the Phase 1 audit |
| Git | 2.55.0.windows.5 | `git --version` |
| Node.js | v24.21.0 | `node --version` |
| npm | 11.19.0 | `npm --version` |
| Python | 3.14.7 | `python --version` |
| .NET SDK | 10.0.401 | `dotnet --version` |
| OpenCode | v2.2.4 | `opencode-cli.exe --version`, `GET /api/info` |

Not available and not to be relied on: `pnpm` and `yarn` (global shims are
broken), Bun (absent), `sqlite3` CLI (absent), Go toolchain (absent), `opencode`
on `PATH` (the CLI is installed only under the desktop application directory,
and a stale 1.14.22 copy also exists).

## Application stack, accepted for the build

| Layer | Choice | Status |
|---|---|---|
| Language | TypeScript, strict mode | Accepted, ADR-010; implementation planned for Phase 2 |
| Framework | Next.js App Router | Accepted, ADR-010; implementation planned for Phase 2 |
| Package manager | npm workspaces | Accepted, ADR-011 |
| Styling | Tailwind CSS v4 | Accepted, ADR-010; implementation planned for Phase 2 |
| Components | shadcn/ui | Accepted, ADR-010; implementation planned for Phase 3, compatibility to be confirmed |
| Icons | Lucide | Accepted, ADR-010; implementation planned for Phase 3 |
| Charts | Recharts | Accepted, ADR-010; implementation planned for Phase 3 |
| Source access | SQLite through a read-only connection | Accepted, ADR-003 |
| Dashboard store | Separate SQLite database with its own migrations | Accepted, ADR-004 |
| SQLite client | `node:sqlite` from Node 24, subject to a Windows spike | Proposed, ADR-012 |
| Validation | Schema validation at every external and data boundary | Planned, Phase 2 |
| Tests | Unit and integration suite for the ingestion, dedup and cost layers | Planned, Phase 2 |

## Source data reality

| Item | Value |
|---|---|
| OpenCode database | `%USERPROFILE%\.local\share\opencode\opencode.db`, override with `OPENCODE_DB` |
| Size at audit time | 2,334,752,768 bytes |
| Journal mode | WAL, live while OpenCode runs |
| Layout | V2 (`session_v2`, `session_message`) plus a completed-migration legacy snapshot (`session`, `message`, `part`) |
| Local server | OpenCode service on loopback, authenticated |

## Not in use

No database server, no container runtime, no cloud service, no telemetry
endpoint and no hosted deployment exist for this project.
