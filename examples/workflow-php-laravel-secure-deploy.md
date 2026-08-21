# Multi-Agent Workflow: PHP/Laravel System — Build, Secure, Deploy to a LAN Server

> The same "build it, secure it, ship it" squad from [workflow-fullstack-build-secure-deploy.md](./workflow-fullstack-build-secure-deploy.md), retargeted at a PHP 8+/Laravel/MySQL stack deployed to a single LAN server (Laragon/XAMPP-style) instead of cloud infrastructure.

## The Scenario

You're building an internal records/ops system for a small government or agency team — think case tracking, staff records, or a risk register — on PHP 8+, Laravel, MySQL, and a Filament or Livewire admin UI. It runs on a single Windows server on the organization's own LAN, not a cloud provider. It still needs a real architecture, a security pass before staff data touches it, and a deployment process that survives being handed off to whoever's on IT duty next.

## Agent Team

| Agent | Role in this workflow |
|-------|---------------------|
| [Software Architect](../engineering/engineering-software-architect.md) | Define module boundaries and the core technical decisions (single Laravel app vs. modular monolith, auth strategy, audit trail design) |
| [Senior Developer](../engineering/engineering-senior-developer.md) | Build the Laravel/Livewire application end to end |
| [Database Optimizer](../engineering/engineering-database-optimizer.md) | Design the MySQL schema, indexes, and query performance for the record volumes involved |
| [Application Security Engineer](../security/security-appsec-engineer.md) | Threat-model the design and review the PHP code for injection, auth, and upload vulnerabilities |
| [Penetration Tester](../security/security-penetration-tester.md) | Attack the staged build on the LAN before real records touch it |
| [DevOps Automator](../engineering/engineering-devops-automator.md) | Script the deployment to the LAN server and the backup/update process |
| [SRE](../engineering/engineering-sre.md) | Set up monitoring, log review, and backup verification for a single-server deployment |
| [Reality Checker](../testing/testing-reality-checker.md) | Gate the launch with evidence, not vibes |

## Install This Squad Locally

```bash
git clone https://github.com/dullahngosha/agency-agents.git
cd agency-agents

./scripts/install.sh --tool claude-code --agent \
  software-architect,senior-developer,database-optimizer,application-security-engineer,penetration-tester,devops-automator,sre-site-reliability-engineer,reality-checker
```

## The Workflow

### Phase 1: Architecture + Threat Model (parallel)

**Step 1 — Activate Software Architect**

```
Activate Software Architect.

System: internal case-tracking system for a ~20-person government office.
Stack: PHP 8.2, Laravel 11, MySQL 8, Livewire + Filament admin panel.
Deployment target: single Windows server on the office LAN (Laragon-style
environment), no internet-facing access, no cloud services.
Requirements: role-based access (staff/supervisor/admin), full audit
history on every record change, file attachments, exportable PDF reports.

Propose the architecture: module boundaries, auth strategy, and how the
audit trail is enforced at the model layer (not just in controllers).
Write it as an ADR.
```

**Step 2 — Activate Application Security Engineer (in parallel)**

```
Activate Application Security Engineer.

Architecture: [paste Software Architect output]
Context: this runs on a closed office LAN, but staff records and audit
logs are still sensitive, and "internal only" is not a security control.

Threat-model this Laravel app before implementation:
1. Where authorization must be enforced (Laravel policies vs. gates vs.
   Filament resource-level access) for every role
2. SQL injection and mass-assignment risks specific to Eloquent/Filament
3. File upload handling (attachments) and MySQL credential storage on a
   shared LAN server
4. What "audit trail" must guarantee (append-only, tamper-evident)

Output a threat model the Senior Developer and Database Optimizer design against.
```

### Phase 2: Build (parallel)

**Step 3 — Activate Database Optimizer**

```
Activate Database Optimizer.

Architecture + threat model: [paste Phase 1 outputs]

Design the MySQL schema:
1. Tables for records, roles/permissions, and an append-only audit log
   (record changes must be reconstructable, not just "last state")
2. Indexes for the expected query patterns (staff searching by case
   number, date range, status)
3. Foreign key and cascade rules that protect the audit trail from
   accidental deletes

Deliver the schema as Laravel migrations.
```

**Step 4 — Activate Senior Developer**

```
Activate Senior Developer.

Architecture: [paste Software Architect output]
Schema: [paste Database Optimizer output]
Threat model: [paste Application Security Engineer output]

Build the Laravel app:
- Livewire components for record list/detail/edit
- Filament admin panel for supervisor/admin roles
- Policy classes enforcing the role rules from the threat model
- File attachment upload with the validation the threat model requires

Start with the record detail view with visible audit history — it's the
screen that proves the audit trail actually works end to end.
```

### Phase 3: Security Hardening

**Step 5 — Activate Application Security Engineer again (code review)**

```
Activate Application Security Engineer.

Implemented code: [paste key policy classes, controllers/Livewire
components handling records and file uploads]

Review against the Phase 1 threat model:
1. Is every Eloquent query using parameter binding — no raw string
   interpolation into SQL anywhere?
2. Are Filament resources scoped by policy, not just hidden from the nav menu?
3. Mass assignment: are $fillable/$guarded set correctly on every model
   touching user input?
4. Is the audit log write path unreachable from any code path that
   bypasses it (e.g. bulk updates, artisan tinker in production)?

Flag anything that must be fixed before staging.
```

**Step 6 — Activate Penetration Tester on the staged LAN build**

```
Activate Penetration Tester.

Target: staging copy of the case-tracking system on the office LAN at
[staging URL/IP]. Scope: authenticated testing across all three roles
(staff/supervisor/admin), focused on privilege escalation between roles,
IDOR on record IDs, and file upload abuse.
Rules of engagement: staging environment only, test accounts provided,
no testing against the production MySQL instance.

Attempt to view, edit, or export records outside your role's scope.
Report findings with reproduction steps and severity.
```

### Phase 4: Deploy to the LAN Server

**Step 7 — Activate DevOps Automator**

```
Activate DevOps Automator.

App: Laravel case-tracking system, cleared by Application Security
Engineer and Penetration Tester.
Target: single Windows server on the office LAN, no cloud, no CDN,
IT staff will maintain this after handoff.

Build:
1. A repeatable deployment script (git pull, composer install --no-dev,
   migrate, cache config/routes/views) that a non-developer can run
2. A scheduled MySQL backup job to a separate drive/share, with a
   documented restore procedure
3. .env and credential handling that keeps secrets out of the repo and
   off shared network drives
4. A short deployment runbook the next IT person can follow without you
```

### Phase 5: Production Reliability + Launch Gate

**Step 8 — Activate SRE**

```
Activate SRE.

Deployment: [paste DevOps Automator output]

For a single-server LAN deployment (no cloud monitoring stack available), set up:
1. Laravel log monitoring (storage/logs) with a simple alert path (e.g.
   email on error-level entries) since there's no APM service
2. A weekly check that confirms backups actually restore, not just that
   the backup job ran
3. Disk space and MySQL health checks appropriate for unattended server hardware
4. A one-page "system is down" runbook for office IT staff
```

**Step 9 — Final Reality Check**

```
Activate Reality Checker.

The case-tracking system is ready to go live for the office.

- Threat model addressed: [paste Application Security Engineer sign-off]
- Pen test findings: [paste Penetration Tester report — all high/critical resolved?]
- Deployment runbook + backup restore tested: [paste DevOps Automator summary]
- Monitoring + backup verification: [paste SRE summary]

Run through the launch checklist and give a GO / NO-GO decision.
Require evidence for each criterion — a backup job that has never been
restore-tested does not count as "backups working."
```

## Key Patterns

1. **"It's on a closed LAN" is not a security control**: the threat model in Phase 1 treats the app the same as an internet-facing one — staff records are still sensitive, and internal threats (a compromised workstation, a disgruntled user) are real.
2. **Audit trail is a database-and-policy problem, not a UI problem**: Database Optimizer designs it as append-only at the schema level; Application Security Engineer verifies no code path can bypass it.
3. **Deployment must survive a handoff**: DevOps Automator's runbook assumes the next person maintaining this system is not the one who built it — this is standard for small government/office teams with staff turnover.
4. **Backups aren't verified until they're restored**: SRE's Phase 5 check is explicit about testing restores, not just confirming the backup job exits with status 0.

## Tips

- If the office later moves this off the LAN to a hosted VM, DevOps Automator's script and SRE's monitoring setup both need a fresh pass — LAN assumptions (no public exposure, physical access control) don't carry over.
- For a smaller system, Software Architect and Senior Developer can be the same activation — don't force a split that doesn't fit the project's size.
- See [workflow-fullstack-build-secure-deploy.md](./workflow-fullstack-build-secure-deploy.md) for the cloud/general-stack version of this same squad structure.
