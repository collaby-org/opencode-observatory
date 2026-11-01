# System diagram

## What runs today

Phase 1 is an audit. The only runtime component is the OpenCode application
itself, plus the read-only inspection the audit performed.

```mermaid
flowchart LR
  OpenCode["OpenCode v2.2.4<br/>runs locally, writes its own database"]
  SourceDb[("OpenCode source database<br/>2.33 GB, WAL mode<br/>read only")]
  Audit["Phase 1 audit scripts<br/>mode=ro + query_only"]
  Reports["docs/*-audit.md<br/>ADR + traceability"]

  OpenCode --> SourceDb
  Audit -->|read only, no writes| SourceDb
  Audit --> Reports
```

## Planned architecture

Accepted in [`architecture-decision-record.md`](architecture-decision-record.md).
Everything below is planned and not implemented yet.

```mermaid
flowchart TB
  subgraph Local["Developer PC, local only"]
    OpenCode["OpenCode<br/>writes its own database"]
    OpenCodeAPI["OpenCode HTTP API<br/>installed local service<br/>optional supplement only"]
    Source[("OpenCode source DB<br/>legacy + V2 layouts<br/>READ ONLY")]

    subgraph Collector["Ingestion layer"]
      Discovery["Source discovery<br/>OPENCODE_DB, XDG_DATA_HOME, default path"]
      Adapters["Source adapters<br/>runtime schema detection<br/>legacy + V2"]
      Worker["Ingestion worker<br/>checkpoints, idempotent upserts,<br/>resume, rebuild, dedup"]
    end

    subgraph Store["Dashboard owned store"]
      Observatory[("Observatory database<br/>raw + derived tables,<br/>own migrations")]
    end

    subgraph App["Local application"]
      API["Local API layer<br/>loopback only,<br/>Host and Origin checks,<br/>no general API proxy"]
      Analytics["Analytics + cost ledgers<br/>six separate ledgers,<br/>versioned pricing catalogue"]
      UI["Next.js interface<br/>overview, explorer, timeline,<br/>models and costs, intelligence"]
      Intelligence["Optional intelligence<br/>adapters, opt-in, budgets,<br/>redaction, typed output"]
    end
  end

  OptionalCloud["Optional hosted mode, future only<br/>explicit, authenticated, sanitised,<br/>never reads the PC database"]

  OpenCode --> Source
  Discovery --> Source
  Adapters -->|read only| Source
  Discovery --> Adapters
  Worker --> Adapters
  Worker -->|writes derived data only| Observatory
  Adapters -.->|optional supplement,<br/>explicit GET allowlist only| OpenCodeAPI
  Analytics --> Observatory
  API --> Analytics
  UI -->|server side only| API
  Intelligence -.->|only after explicit opt-in| Observatory
  Observatory -.->|future, opt-in sync| OptionalCloud
```

## Boundaries that must hold

- The source database is opened read-only and is never migrated, vacuumed or
  written.
- The OpenCode HTTP API is used only through an explicit GET allowlist, never
  proxied.
- The local API layer binds loopback, validates Host headers and applies Origin
  checks to its state-changing routes; it is not limited to GET.
- The dashboard database can be deleted and rebuilt without touching the
  source.
- The local server binds loopback by default and rejects foreign Host headers.
- AI providers receive nothing until the user configures a model and asks for a
  summary, and only after redaction.
