# Multi-Agent Workflow: Full-Stack Build, Security, and Deployment

> Coordinate 7 agents to take a system from architecture through secure, monitored production deployment — the standard "build it, secure it, ship it" squad.

## The Scenario

You're building an internal system (e.g. an admin dashboard, a records/ops platform) that needs to go from a blank repo to a live, secured, monitored production deployment. Nobody signs off on "it runs on my machine" — the system needs an architecture that will scale, a security pass before anything real touches it, and a deployment pipeline that doesn't depend on someone remembering the steps.

## Agent Team

| Agent | Role in this workflow |
|-------|---------------------|
| [Software Architect](../engineering/engineering-software-architect.md) | Define the system architecture, data boundaries, and key technical decisions |
| [Backend Architect](../engineering/engineering-backend-architect.md) | Design the API contract and database schema |
| [Frontend Developer](../engineering/engineering-frontend-developer.md) | Build the UI against the API contract |
| [Application Security Engineer](../security/security-appsec-engineer.md) | Threat-model the design and review code for vulnerabilities before launch |
| [Penetration Tester](../security/security-penetration-tester.md) | Attack the staged build before it goes live |
| [DevOps Automator](../engineering/engineering-devops-automator.md) | Build the CI/CD pipeline and provision infrastructure |
| [SRE](../engineering/engineering-sre.md) | Wire up monitoring, alerting, and rollback before go-live |
| [Reality Checker](../testing/testing-reality-checker.md) | Gate the launch with evidence, not vibes |

## Install This Squad Locally

These agents already exist in this repo — no new agent files needed. To install just this squad on your own machine (not in this cloud session — `scripts/install.sh` writes to your local tool config, so run it where the tool actually lives):

```bash
git clone https://github.com/dullahngosha/agency-agents.git
cd agency-agents

# Claude Code: writes to ~/.claude/agents/
./scripts/install.sh --tool claude-code --agent \
  software-architect,backend-architect,frontend-developer,application-security-engineer,penetration-tester,devops-automator,sre-site-reliability-engineer,reality-checker
```

Swap `--tool claude-code` for `codex`, `cursor`, `gemini-cli`, `opencode`, `windsurf`, `qwen`, `zcode`, `copilot`, `vibe`, `aider`, `antigravity`, `osaurus`, `openclaw`, or `hermes` — or `--tool all` to install to every tool `install.sh` detects on your machine. Add `--dry-run` first to preview what would be written, or `--link` to symlink instead of copy so future `git pull`s update the agents in place. Run `./scripts/install.sh --help` for the full option list.

## The Workflow

### Phase 1: Architecture + Threat Model (parallel)

**Step 1 — Activate Software Architect**

```
Activate Software Architect.

System: internal case-management platform for a small operations team (10-30 staff).
Requirements: user auth with roles (staff/admin), record CRUD with audit history,
file attachments, search, exportable reports.
Constraints: small team maintaining it long-term, must run on modest infra
(single VM or small container cluster — no exotic managed services).

Propose the architecture: module boundaries, data flow, and the 2-3 biggest
technical decisions with trade-offs. Write it as an ADR.
```

**Step 2 — Activate Application Security Engineer (in parallel)**

```
Activate Application Security Engineer.

Here's the proposed architecture: [paste Software Architect output]

Threat-model this before a line of code is written:
1. Trust boundaries and where auth/authorization must be enforced
2. Sensitive data (PII, audit logs) and how it must be handled at rest and in transit
3. The top risks for this shape of system (role-based access, file uploads, audit trail)

Output a short threat model the Backend Architect and Frontend Developer
must design against.
```

### Phase 2: Build (parallel)

**Step 3 — Activate Backend Architect**

```
Activate Backend Architect.

Architecture: [paste Software Architect output]
Threat model: [paste Application Security Engineer output]

Design the API and database schema:
1. Database schema with audit-history tracking built in
2. REST API endpoints, with authorization rules per endpoint
3. File upload handling that respects the threat model's constraints

Deliver schema (SQL) + endpoint list + auth strategy.
```

**Step 4 — Activate Frontend Developer**

```
Activate Frontend Developer.

API spec: [paste Backend Architect output]

Build the UI:
- Role-aware views (staff vs admin)
- Record list/detail/edit with audit history visible
- File upload + search

Start with the record detail view — it's the core screen everything else supports.
```

### Phase 3: Security Hardening

**Step 5 — Activate Application Security Engineer again (code review)**

```
Activate Application Security Engineer.

Here's the implemented backend: [paste API/auth code or a summary of it]
Here's the implemented frontend: [paste key auth/data-handling code]

Review against the threat model from Phase 1:
1. Are authorization checks enforced server-side on every endpoint?
2. Any injection, upload, or IDOR risks in what was built?
3. Secrets/credentials handling — anything hardcoded or logged?

Flag anything that must be fixed before this goes to staging.
```

**Step 6 — Activate Penetration Tester on staging**

```
Activate Penetration Tester.

Target: staging deployment of the case-management platform at [staging URL].
Scope: authenticated + unauthenticated testing of auth, authorization
boundaries between staff/admin, file upload handling, and API endpoints.
Rules of engagement: staging environment only, test accounts provided, no
destructive testing against shared data.

Attempt to break role boundaries and access records/files you shouldn't be
able to. Report findings with reproduction steps and severity.
```

### Phase 4: Deploy

**Step 7 — Activate DevOps Automator**

```
Activate DevOps Automator.

App: case-management platform (backend + frontend from Phases 2-3), now
cleared by Application Security Engineer and Penetration Tester.
Target: single small VM or container cluster, budget-conscious.

Build:
1. CI/CD pipeline (build, test, security scan, deploy)
2. Infrastructure as code for the target environment
3. Zero-downtime deploy strategy and a documented rollback path
4. Secrets management (no credentials in the repo or pipeline logs)
```

### Phase 5: Production Reliability + Launch Gate

**Step 8 — Activate SRE**

```
Activate SRE.

Deployment pipeline: [paste DevOps Automator output]

Before this takes real traffic, set up:
1. Uptime and error-rate monitoring with alert thresholds
2. Log aggregation for the audit trail and error diagnostics
3. Automated backup schedule for the database
4. A one-page runbook for "it's down, now what"
```

**Step 9 — Final Reality Check**

```
Activate Reality Checker.

The case-management platform is ready to launch. Evaluate production readiness:

- Threat model addressed: [paste Application Security Engineer sign-off]
- Pen test findings: [paste Penetration Tester report — all high/critical resolved?]
- CI/CD + rollback path: [paste DevOps Automator summary]
- Monitoring + backups: [paste SRE summary]

Run through the launch checklist and give a GO / NO-GO decision.
Require evidence for each criterion — no unresolved high-severity findings.
```

## Key Patterns

1. **Security runs twice, not once**: threat-modeled *before* the build (Phase 1) and adversarially tested *after* it (Phase 3) — a single security pass at the end catches design flaws too late to fix cheaply.
2. **Build and secure in the same loop**: Application Security Engineer reviews real code, not just the architecture doc — vulnerabilities live in implementation details the design phase can't see.
3. **Deployment is not the last step**: SRE's monitoring/backup/rollback setup happens *before* go-live, not after the first incident.
4. **Reality Checker requires evidence**: the launch gate demands the actual pen-test report and security sign-off, not a verbal "looks good."

## Tips

- Don't skip Phase 3's second Application Security Engineer pass — code drifts from the Phase 1 design during implementation.
- If the Penetration Tester finds a high/critical issue, loop back to Backend Architect or Application Security Engineer to fix it, then re-test before Phase 4.
- For a smaller system, Software Architect and Backend Architect can be the same activation — don't force a split that doesn't fit the project's size.
- Keep the [Agents Orchestrator](../specialized/agents-orchestrator.md) in mind for automating this pipeline once you've run it manually a few times.
