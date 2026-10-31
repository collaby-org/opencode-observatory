# Project doc

## What it is

OpenCode Observatory is a local-first web application that turns the complete
local session history of OpenCode into an auditable analytics and
project-intelligence dashboard. It runs on the developer's own Windows machine,
reads the OpenCode database without ever modifying it, keeps every derived
record in its own database, and explains where each number comes from.

## Problem

OpenCode keeps a rich local record of what happened, but that record is a
database, not a report. There is no way to answer "what did this project cost",
"how much of this was free-tier usage", "which sessions never finished", or
"why does this file keep changing" without writing SQL by hand. Existing
viewers answer parts of this, but they either target the retired V1 schema,
write to the source database, or present a single dollar number that mixes
measured spend with estimates.

## Who it is for

A developer who uses OpenCode across many repositories, worktrees, models and
subagents, and who has accumulated thousands of sessions in a local database.
They want an overview for leadership-level questions, a session explorer for
forensics, and project intelligence for continuity, all without uploading
private transcripts to a service.

## Why it has not stuck before

The obvious approach is to query the OpenCode database directly from a web
server. It is fast to build and wrong in three ways: it assumes one schema, it
has nowhere to keep checkpoints or derived analytics, and it makes every
number a re-computation with no provenance. The other obvious approach is to
reuse an existing open-source dashboard. The three candidates audited in Phase 1
are all instructive precisely because each fails differently: one writes to the
source database, one targets the retired schema and exposes an unauthenticated
server on every network interface, and one is a strong Go implementation that
ships no Windows binary and carries a scope far wider than this product.

## The rule everything rests on

Every number shows its source. If a value cannot be traced to a record, a rate
card or a labelled estimate, it is shown as unknown rather than as a number.

## Scope for the first release

A local Windows application with an overview, a project and session explorer, an
agent activity timeline, model and token analytics, provenance-aware cost
ledgers with a free-usage counterfactual, optional evidence-backed project
intelligence, reliability views, local search, export and diagnostics.

## Out of scope, and why

- **Modifying or controlling OpenCode sessions.** The source database is the
  user's record of work, not an input to be edited.
- **Running commands found in transcripts.** Historical content is untrusted
  data, never an instruction.
- **Automatic cloud processing of history.** Transcripts can contain source
  code, prompts and credentials.
- **A multi-user hosted dashboard.** The first release serves one developer on
  one machine.
- **Exact actual charges without billing evidence.** Only an authoritative
  invoice or billing API can support that claim.
- **A desktop extension** before OpenCode's extension points are verified.

## How it works

A local server discovers the OpenCode database through documented paths, opens
it with a read-only SQLite connection, detects the schema and migration state
at runtime, and imports changed records into a dashboard-owned database through
a checkpointed, idempotent ingestion worker. The dashboard reads only that
database. Costs are kept in six separate ledgers, each amount carrying its
source, rate card, quality and calculation time. AI features are adapters
behind an explicit opt-in, validated against a typed schema and linked to
evidence.

## Interface

A local browser application with a left navigation for Overview, Projects,
Sessions, Agent Activity, Models and Costs, Project Intelligence, Files and
Changes, Reliability, and Data Sources and Settings. A top bar carries global
search, the date range, source and sync status, and settings.

## Where it is likely to go wrong

- **Schema drift.** OpenCode changes its schema between versions, and a
  migrated database keeps two layouts at once.
- **Double counting.** Migrated history, forked sessions and 1,085 parent-linked
  sessions in the audited database all create ways to count the same work
  twice.
- **Cost confusion.** A stored cost and an API-equivalent estimate look alike
  on screen and must never be added together.
- **Silent zeros.** A failed request with no usage recorded must read as
  unavailable, not as free.
- **Privacy leaks.** Session text, file paths and tool output can contain
  secrets.

## Ship criteria

A release is acceptable only when the source database hash and schema are
unchanged by a full run, two consecutive syncs produce identical counts, cost
values are separated and auditable, the four core experiences work on real
imported data, AI insights link to evidence, the automated suite passes on
Windows, the installation and recovery guide works on a clean machine, and the
known limitations are written down honestly.
