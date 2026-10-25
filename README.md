# PROJECT_NAME

> A production-ready starter template for modern web applications.

## Overview

This template provides a complete foundation for building production web applications with:

- **Agent workflow** — Structured development process via `AGENTS.md`
- **GitHub Actions CI/CD** — SHA-pinned workflows for CI, secret scanning, code review
- **Automated reviews** — Kodus, PR-Agent, and opencode integration
- **Design system** — Tailwind CSS + shadcn/ui + Lucide React ready
- **Documentation standards** — Living docs, changelogs, versioning protocol

## Quick Start

### 1. Use this template

Click **"Use this template"** on GitHub, or clone and push to a new repository:

```bash
git clone https://github.com/YOUR_USERNAME/starter-kit.git my-new-project
cd my-new-project
# Remove template git history and start fresh
rm -rf .git
git init
git add .
git commit -m "feat: initial commit from starter-kit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/my-new-project.git
git push -u origin main
```

### 2. Configure secrets

Add the following secrets in **Settings → Secrets and variables → Actions**:

| Secret | Required | Description |
|--------|----------|-------------|
| `LLM_API_KEY` | Yes | NVIDIA API key for LLM access |
| `LLM_API_BASE` | Yes | `https://integrate.api.nvidia.com/v1` |
| `KODUS_API_URL` | No | Kodus review API endpoint |
| `KODUS_TEAM_KEY` | No | Kodus team authentication key |
| `GITHUB_TOKEN` | Auto | Provided automatically by GitHub Actions |

See `PRIVATE_CONFIG.md` for details (never committed).

### 3. Customize for your project

- Update `PROJECT_NAME` references in `UI_UX_MASTER.md` and `UI_UX_PROMPTS.md`
- Adjust `dependabot.yml` directory paths to match your structure
- Update `ci.yml` job configurations for your stack
- Modify `CODEOWNERS` with your team handles
- Replace `SECURITY-CONTACT-PLACEHOLDER` in `SECURITY.md`

## Project Structure

```
├── .github/
│   ├── workflows/       # CI/CD pipelines (SHA-pinned)
│   ├── ISSUE_TEMPLATE/  # Issue forms
│   ├── CODEOWNERS       # Review assignments
│   ├── dependabot.yml   # Dependency updates
│   └── SECURITY.md      # Security policy
├── design-system/       # UI tokens, components, patterns
├── docs/                # Living documentation
│   ├── changelog.md
│   ├── roadmap.md
│   ├── tech_stack.md
│   └── ui-ux/           # UI/UX migration artifacts
├── AGENTS.md            # Global agent workflow (source of truth)
├── CLAUDE.md            # Points to AGENTS.md
├── CONTRIBUTING.md      # Contribution guidelines
├── COMMANDS.md          # Agent command reference
├── UI_UX_MASTER.md      # Master redesign specification
├── UI_UX_PROMPTS.md     # Step-by-step orchestrator prompts
├── opencode.json        # OpenCode configuration
├── skills-lock.json     # Pinned agent skills
└── PRIVATE_CONFIG.md    # Secrets reference (gitignored)
```

## Agent Workflow

Every task follows four beats (see `AGENTS.md`):

1. **Isolate** — Worktree from `origin/main`, never build on `main`
2. **Build** — `/develop` with stack skills, `/architect` for undecided decisions
3. **Prove** — Tests, `/check verify`, Kodus loop, commit, rebase, local code review
4. **Ship** — PR with evidence, `/document`, `/unslop`, PR-Agent + Kodus to zero

### Key Commands

| Command | Purpose |
|---------|---------|
| `/scope` | Create living plan |
| `/audit` | Bootstrap AGENTS.md (human-only) |
| `/architect` | Write spec in `docs/specs/` |
| `/new-feature` | Fresh worktree + branch |
| `/develop` | Build from spec |
| `/check verify` | Prove behavior against spec |
| `/check review` | Senior code review (fresh model) |
| `/document` | Draft PR/changelog from diff |
| `/sync` | Reconcile docs with repo (human-only) |

## GitHub Actions

### Core Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | PR, push to main | Consolidated build, lint, test |
| `secret-scan.yml` | PR, push, weekly | Gitleaks secret scanning |
| `kodus-review.yml` | PR opened/sync | Kodus CLI review (advisory) |
| `pr-agent.yml` | PR, `/review` comment | PR-Agent review (advisory) |

### OpenCode Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `opencode.yml` | `/oc` mention | Mention responder |
| `opencode-review.yml` | PR opened/sync | Automatic PR review |
| `opencode-scheduled.yml` | Weekly (Mon 09:23) | Repository health check |
| `opencode-triage.yml` | Issue opened | Issue triage |

All workflows are **credential-gated** — they skip (green, zero minutes) if
required secrets are not configured.

## Documentation Protocol

- **Living docs**: `docs/changelog.md`, `system_diagram.md`, `targets.md`,
  `roadmap.md`, `tech_stack.md`, `project-doc.md`
- **Versioning**: One PR → one bump on the folder's version line
  - Feature/architecture → minor
  - Fix/docs/chore → patch
  - Breaking → major
- **Changelog**: Entry lands in same commit, in folder's changelog
- **Stale docs are a bug** — update at end of every task

## UI/UX Redesign Process

If redesigning a frontend, follow the 7-step process in `UI_UX_MASTER.md`:

1. **Read-only audit** — Map routes, components, CSS, journeys
2. **Design system** — Define tokens, components, migration plan
3. **Representative page** — Prove system on one page
4. **Shared foundation** — Migrate shell and common components
5. **Core product areas** — Migrate dense data interfaces
6. **Journeys & polish** — Onboarding, discovery, motion
7. **Legacy retirement** — Remove old CSS, final QA

Each step requires explicit approval before proceeding.

## License

MIT — Use freely for any project.