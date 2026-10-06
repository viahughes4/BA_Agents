# BA_Agents Quick Reference

## TBA Commands (`.claude/commands/`)

**Setup:**
- `/sow-importer` — Parse SOW/RFP → project-context.md
- `/onboard-project` — Conversational intake (fill gaps)
- `/gap-analysis` — Score spec + AI-codegen readiness

**Discovery & Storytelling:**
- `/feature-discovery [feature]` — 15-dimension analysis (Light/Full)
- `/generate-stories [spec]` — Self-validating Jira stories
- `/generate-prd [scope]` — Client PRD assembly
- `/spec-to-tickets [spec]` — Batch ticket generation

**Validation & QA:**
- `/validate-stories [story]` — 5-lens validation
- `/validate-ac [story]` — AC quality check only
- `/cert-checklist [sprint]` — Platform cert prep

**Reporting & Admin:**
- `/jira-cleanup [weekly|all]` — Spec-readiness + decision-blocked sweeps
- `/status-report [format]` — Formats: delivery, sprint, weekly, exec, release
- `/meeting-helper [transcript]` — Extract actions + decisions
- `/estimate-review [sprint]` — Compare estimates vs. catalog

---

## PM Skills (by domain)

### Discovery
```
/discovery:feature-discovery      Feature breakdown + cross-domain questions
/discovery:gap-analysis            Requirements audit vs. catalog
/discovery:new-feature             New feature intake template
```

### Storytelling
```
/storytelling:generate-stories     Batch story generation (flexible spec input)
/storytelling:spec-to-tickets      Spec → structured ticket breakdown
/storytelling:generate-prd         Extended PRD assembly (multi-format)
/storytelling:validate-stories     Extended validation (external checks)
/storytelling:validate-ac          AC quality + consistency check
```

### Tracking
```
/tracking:weekly-tracker           Weekly status + blockers tracker
/tracking:backlog-grooming         Backlog health + refinement
/tracking:jira-cleanup             Broader Jira hygiene + sweeps
/tracking:ticket-generation        Batch ticket creation + linking
```

### Reporting
```
/reporting:status-report           Multi-format status (internal/client/exec)
/reporting:project-health-report   Health dashboard + velocity
/reporting:release-prep            Release checklist + comms plan
```

### Team-Ops
```
/team-ops:standup-prep             Daily standup briefing
/team-ops:morning-briefing         Morning digest (15-min read)
/team-ops:meeting-helper           Meeting notes → actions
/team-ops:end-of-day               EOD summary + next-day prep
```

### Admin
```
/admin:jira-qa-duplicates          Find + merge duplicate tickets
/admin:escalation-brief            Escalation digest + mitigation
```

---

## Orchestrators (Agents)

**Use when you want a full workflow orchestrated:**

### ba-orchestrator
Routes BA work by phase:
- Setup: sow-importer → onboard-project → gap-analysis
- Feature: feature-discovery → generate-stories → validate-stories
- Periodic: jira-cleanup, status-report

Trigger: `"route this BA work"`, `"what should I do next"`

### discovery-agent
5-phase OTT discovery (raw SOW → execution-ready):
1. Intake: sow-importer + onboard-project
2. Gap analysis
3. Feature discovery
4. Story generation
5. Validation + cert prep

Trigger: `"run discovery"`, `"start discovery"`, `"/discovery"`

### intake-orchestrator
Raw input → structured work:
- Routes to: sow-importer, gap-analysis, spec-to-tickets, ticket-generation
- Outputs: project-context.md + tickets in Jira

Trigger: `"intake this"`, `"turn this into tickets"`, `"ticket this up"`

### project-kickoff-agent
Pre-sales → execution-ready (4-phase):
1. SOW parsing + context
2. Gap analysis + scope
3. Epic breakdown
4. Schedule + team assignments

Trigger: `"run kickoff"`, `"start kickoff"`, `"new project"`

---

## Routing Rules

| Scenario | Use |
|----------|-----|
| OTT project, start fresh | `/onboard-project` or `/discovery-agent` |
| Have SOW/PRD | `/sow-importer` first, then `/gap-analysis` |
| Feature not yet storified | `/feature-discovery` → `/generate-stories` |
| Bulk tickets from spec | `/spec-to-tickets` → `/ticket-generation` |
| Re-validate existing stories | `/validate-stories [jira-key]` |
| Weekly standup prep | `/standup-prep` or `/async-standup` skill |
| Status report (client-facing) | `/status-report exec` or `status-report client` |
| Jira hygiene sweep | `/jira-cleanup all` |
| Cert prep (OTT only) | `/cert-checklist [sprint]` |
| Full discovery workflow | `/discovery-agent` (orchestrates 5 phases) |
| Non-OTT domain | Skip commands, use skills directly |

---

## Common Workflows

### New OTT Engagement
```
1. /onboard-project                  (or sow-importer)
2. /gap-analysis
3. /cert-checklist                   (4-6 weeks before cert)
4. /feature-discovery [features]     (for each non-trivial feature)
5. /generate-stories [spec]          (once feature clear)
6. /validate-stories [jira-key]      (if story edited externally)
```

### Bulk Ticket Generation
```
1. /spec-to-tickets [spec]           (identify features)
2. /ticket-generation [output]       (create Jira tickets)
3. /validate-stories [sprint]        (QA pass before sprint)
```

### Weekly Reporting
```
1. /jira-cleanup weekly              (fix spec-readiness + blockers)
2. /status-report weekly             (internal digest)
3. /status-report exec               (if exec summary needed)
```

### Daily Team Sync
```
/standup-prep                        (or async-standup skill)
/meeting-helper [transcript]         (after meetings)
/end-of-day                          (wrap + next-day prep)
```

---

## Duplication Handling

Some skills exist in both commands and the PM skills suite. Here's how they compose:

| Capability | Command | Skill | Use Command When | Use Skill When |
|------------|---------|-------|------------------|----------------|
| feature-discovery | ✓ (OTT-specific 15D) | ✓ (general) | OTT domain, catalog-driven | Non-OTT, or expanding OTT discovery |
| generate-stories | ✓ (self-validating) | ✓ (batch generation) | Single feature or sprint | Bulk generation from spec |
| validate-stories | ✓ (5-lens OTT) | ✓ (extended + external) | Self-validation after gen | External QA or re-validation |
| gap-analysis | ✓ (OTT catalog scoring) | ✓ (general audit) | OTT project scoping | General domain requirements audit |
| jira-cleanup | ✓ (spec-readiness focused) | ✓ (broader hygiene) | Targeted cleanup | Full sweep + automation |
| status-report | ✓ (TBA delivery snapshot) | ✓ (multi-format: exec/client/internal) | OTT status | Other formats or non-OTT |
| meeting-helper | ✓ (TBA-focused signal) | ✓ (general digest) | OTT standups | Other meeting types |

**Rule:** Commands = OTT-specific depth. Skills = cross-domain breadth.

