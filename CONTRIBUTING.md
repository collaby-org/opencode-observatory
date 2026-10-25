# Contributing

Read [`AGENTS.md`](AGENTS.md) first. It is the workflow every contribution
follows. The project guide — the stacks, the folders and the per-tree
commands — is [`\.github/CONTRIBUTING.md`](.github/CONTRIBUTING.md). This file
covers only what a change must contain.

## One problem per pull request

A pull request solves one problem. A refactor, a dependency bump and a bug fix
are three pull requests. If the work needs a second problem solved to land,
land the first one and open the second one separately.

## Conventional commits

Every commit uses a conventional commit message: `feat:`, `fix:`, `docs:`,
`chore:`, `refactor:`, `test:` or `build:`. The subject says what changed in
plain words. The body says why, when the reason is not obvious.

## Evidence is required

A pull request states what you ran, what it printed, and what you did **not**
check. Evidence means:

1. The exact commands you ran, copyable as written.
2. The relevant output, pasted — not summarized as "tests pass".
3. The parts you did not verify, named honestly.

Screenshots are welcome for interface changes. They never replace the commands
and output for the logic underneath.

## The pull request body

Fill every section of
[`\.github/pull_request_template.md`](.github/pull_request_template.md),
including the **`version: before → after`** line and the **Verification
evidence**. "None" or "not applicable" is a valid answer when it is true; an
empty section is not.

## Version and changelog

The history lives in [`docs/changelog.md`](docs/changelog.md).

- A feature or architecture change bumps the minor version.
- A fix, a docs change or a chore bumps the patch version.
- A breaking change bumps the major version.
- Every change made while a pull request is open bumps the patch again.

The changelog line lands in the same commit as the change, in the changelog of
the folder that was worked in.

## Flow

1. Run the checks the touched tree owns.
2. Run the Kodus review loop (`/kodus-review`) on the staged diff until it is
   clean.
3. Commit with a conventional message, then rebase on `main` and rerun the
   checks.
4. Run the local code review (`/code-review`) on the final HEAD, on a model
   that did not write the code. A change after the review makes it stale, so
   rerun it.
5. Open the pull request and fill every section of the template.

CI runs [`ci.yml`](.github/workflows/ci.yml) and
[`secret-scan.yml`](.github/workflows/secret-scan.yml) on every pull request.
The Kodus and PR-Agent workflows are advisory and skip themselves when a
repository has no secrets configured.

## Review rounds

Two review rounds are the limit. Each round ends with the reviewer's new
findings resolved or explicitly deferred with a reason. After the second
round, stop and fix the cause: split the pull request, change the design, or
ask for a decision. Do not open a third round on the same shape.

## Secrets

Never commit `.env` files, tokens, keys, customer data or private URLs. If a
secret reaches a commit, rotate it first and report it through the private
channel in [`\.github/SECURITY.md`](.github/SECURITY.md), not in the pull
request.