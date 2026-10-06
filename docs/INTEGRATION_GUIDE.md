# BA_Agents Integration Guide

## Two Layers: Commands + Skills

**Mike's TBA Commands** (13 PM-facing entry points in `.claude/commands/`)
- `/sow-importer`, `/onboard-project`, `/feature-discovery`, `/generate-stories`, `/generate-prd`
- `/validate-stories`, `/validate-ac`, `/jira-cleanup`, `/cert-checklist`, `/estimate-review`
- `/status-report`, `/meeting-helper`, `/gap-analysis`

**Vanessa's Skills** (23+ reusable units in `.claude/skills/` by domain)
- Discovery: feature-discovery, gap-analysis, new-feature
- Storytelling: generate-stories, spec-to-tickets, generate-prd, validate-stories, validate-ac
- Tracking: weekly-tracker, backlog-grooming, jira-cleanup, ticket-generation
- Reporting: status-report, project-health-report, release-prep
- Team-ops: standup-prep, morning-briefing, meeting-helper, end-of-day
- Admin: jira-qa-duplicates, escalation-brief

---

## How They Interact

### Mike's Commands Call Your Skills

| Command | Invokes Skills |
|---------|----------------|
| `/sow-importer` | Internal (catalog-based) |
| `/onboard-project` | Internal (catalog-based) |
| `/feature-discovery` | Internal (OTT-specific logic) |
| `/generate-stories` | Links to your `spec-to-tickets` for batch generation |
| `/validate-stories` | Uses your `validate-stories` skill for extended validation |
| `/gap-analysis` | Uses your `gap-analysis` skill for domain expansion |
| `/generate-prd` | Links to your `generate-prd` skill for client-facing PRD assembly |
| `/jira-cleanup` | Uses your `jira-cleanup` skill for spec-readiness sweeps |
| `/status-report` | Uses your `status-report` skill for client-facing variants |
| `/meeting-helper` | Uses your `meeting-helper` skill for standup synthesis |

### Agents Orchestrate Commands + Skills

**ba-orchestrator** — routes BA work by workflow phase
- Setup: sow-importer → onboard-project → gap-analysis
- Feature work: feature-discovery → generate-stories → validate-stories
- Periodic: jira-cleanup, status-report (runs your skills directly)

**discovery-agent** — 5-phase OTT discovery
- Calls `/onboard-project` + `/gap-analysis` for intake
- Chains into feature-discovery → generate-stories → validate-stories

**intake-orchestrator** — turns raw input into structured work
- Runs feature-discovery, spec-to-tickets, generates tickets
- Uses your skill suite for ticket generation + validation

**project-kickoff-agent** — pre-sales to execution-ready
- Calls sow-importer, onboard-project, gap-analysis
- Chains to spec-to-tickets, ticket-generation

---

## Routing Logic (CLAUDE.md)

```
User says "discover this feature"
  → ba-orchestrator picks: light feature-discovery (skip if trivial)
  → If non-trivial: run full feature-discovery
  → Call your skill: `/generate-stories` via spec-to-tickets
  → Then: `/validate-stories` for quality gate

User says "run discovery"
  → discovery-agent picks: 5-phase OTT workflow
  → Calls Mike's commands: sow-importer, onboard-project, gap-analysis
  → Then chains into feature-discovery + your skills

User says "intake this SOW"
  → intake-orchestrator picks: sow-importer → gap-analysis
  → Routes to: spec-to-tickets, ticket-generation
  → Uses your skill suite for artifact creation
```

---

## No Duplication

- **feature-discovery** exists in both Mike's commands AND your skills
  - Mike's command = OTT-specific 15-dimension report (catalog-driven)
  - Your skill = expanded discovery with external integrations
  - Routing: Mike's command runs first; your skill fills gaps
  
- **generate-stories** exists in both
  - Mike's command = self-validating Jira-ready output (catalog + context)
  - Your skill = batch generation from any spec (more flexible)
  - Routing: Mike's for single features; your skill for bulk generation

- **validate-stories** exists in both
  - Mike's sub-agent = 5-lens OTT validation (part of generate-stories)
  - Your skill = re-validation + external checks (standalone)
  - Routing: Mike's for self-validation; your skill for external verification

- **gap-analysis** exists in both
  - Mike's command = catalog scoring + AI-codegen readiness (OTT only)
  - Your skill = general domain gap analysis (cross-domain)
  - Routing: Mike's for OTT; your skill for non-OTT projects

- **jira-cleanup** exists in both
  - Mike's command = spec-readiness + decision-blocked sweeps (command-driven)
  - Your skill = broader Jira hygiene (automation-focused)
  - Routing: Mike's for targeted cleanup; your skill for sweeps

- **status-report** exists in both
  - Mike's command = TBA-specific delivery snapshot (catalog context)
  - Your skill = multi-format reporting (exec/client/internal variants)
  - Routing: Mike's for OTT status; your skill for other formats

- **meeting-helper** exists in both
  - Mike's command = TBA-focused signal extraction from transcripts
  - Your skill = general meeting digest + action items
  - Routing: Mike's for OTT standups; your skill for other meetings

---

## When to Use Each

**Use Mike's TBA Commands:**
- You're on an OTT/streaming project with catalog-driven scope
- You need AI-codegen spec readiness validation
- You're running the full discovery → stories → validate cycle
- You need certification prep (cert-checklist is OTT-specific)

**Use Your Skills:**
- You're on a non-OTT domain
- You need batch operations (backlog-grooming, ticket-generation, jira-cleanup sweeps)
- You want extended validation (validate-stories with external checks)
- You need reporting variants (exec brief, client status, health report)
- You're automating standup/morning briefing/EOD routines

**Use Agents:**
- You want orchestrated workflows (discovery-agent chains commands + skills)
- You need context-aware routing (ba-orchestrator picks the right tool)
- You want end-to-end project setup (project-kickoff-agent)

---

## Team Collaboration

**For PMs:** Start with `/onboard-project` or agents. Commands are entry points.
**For BA tools:** Commands provide the OTT domain logic; skills handle cross-domain patterns.
**For Accedo:** TBA commands = product offering; your skills = delivery operations.

