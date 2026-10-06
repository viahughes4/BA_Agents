# BA_Agents

**Technical Business Analyst + PM Operations toolkit** combining:
- Mike's OTT-specific TBA agent (13 PM commands + 6 sub-agents)
- Vanessa's Accedo PM skills (23+ reusable operations)
- Integrated orchestrators (ba-orchestrator, discovery-agent, intake-orchestrator, project-kickoff-agent)

Used by PMs, BAs, and PM tools to scope, story, validate, and track OTT and general delivery work.

---

## Quick Start

### For OTT Projects

```bash
git clone https://github.com/viahughes4/BA_Agents.git
cd BA_Agents
claude
```

Your first command:
```
/sow-importer        # Parse a client SOW → bootstrap project-context.md
/onboard-project     # Fill in gaps interactively
```

Then use the TBA workflow:
```
/feature-discovery → /generate-stories → /validate-stories
```

### For General PM Work

```bash
# Use agents for orchestration
/ba-orchestrator         # Routes BA work by workflow phase
/discovery-agent         # 5-phase discovery for any domain
/intake-orchestrator     # Raw input → structured tickets
/project-kickoff-agent   # Pre-sales → execution-ready
```

---

## What You Get

### 13 TBA Commands (Mike's foundation)

| Command | Use for |
|---------|---------|
| `/sow-importer` | Parse SOW/PRD → project context |
| `/onboard-project` | Conversational PM intake |
| `/gap-analysis` | Score spec + AI-codegen readiness |
| `/feature-discovery` | 15-dimension feature analysis (Light/Full) |
| `/generate-stories` | Self-validating Jira-ready stories |
| `/generate-prd` | Client-facing PRD assembly |
| `/validate-stories` | 5-lens story validation |
| `/validate-ac` | Single-story AC quality check |
| `/jira-cleanup` | Spec-readiness + decision-blocked sweeps |
| `/cert-checklist` | Platform certification prep |
| `/estimate-review` | Compare estimates vs. catalog benchmarks |
| `/status-report` | Delivery snapshot (multiple formats) |
| `/meeting-helper` | Digest meeting transcripts → actions |

### 23+ Organized Skills (Vanessa's suite)

**Discovery** — feature-discovery, gap-analysis, new-feature
**Storytelling** — generate-stories, spec-to-tickets, generate-prd, validate-stories, validate-ac
**Tracking** — weekly-tracker, backlog-grooming, jira-cleanup, ticket-generation
**Reporting** — status-report, project-health-report, release-prep
**Team-ops** — standup-prep, morning-briefing, meeting-helper, end-of-day
**Admin** — jira-qa-duplicates, escalation-brief

### 4 Orchestrators (from accedo-pm-template)

| Agent | Workflow |
|-------|----------|
| **ba-orchestrator** | Routes BA work (setup → feature → periodic) |
| **discovery-agent** | 5-phase OTT discovery (raw → execution-ready) |
| **intake-orchestrator** | Input → structured Jira tickets |
| **project-kickoff-agent** | Pre-sales → fully structured project |

---

## How It Works

**TBA commands** provide OTT-specific logic (DRM, player SDKs, platform certs, multilingual edge cases).

**Skills** handle general PM operations (trackers, cleanups, reporting, team sync).

**Agents** orchestrate both into workflows.

**No duplication** — when the same capability exists in both (e.g., validate-stories, jira-cleanup), they compose:
- Commands = domain-specific depth (OTT catalog logic)
- Skills = cross-domain breadth (batch ops, extended validation)

See `docs/INTEGRATION_GUIDE.md` for detailed routing and when to use each.

---

## File Structure

```
BA_Agents/
├── README.md                          # Mike's original TBA README
├── README_BA_AGENTS.md                # ← You are here
├── CLAUDE.md                          # TBA agent behavior + integration routing
├── OTT-requirements-reference.md      # Catalog of OTT features/systems/effort
├── project-context.md                 # Per-engagement (gitignored — local only)
├── .mcp.json                          # MCP config (Slack, etc.)
├── .claude/
│   ├── commands/                      # 13 TBA PM-facing commands
│   │   ├── sow-importer.md
│   │   ├── onboard-project.md
│   │   ├── feature-discovery.md
│   │   ├── generate-stories.md
│   │   ├── generate-prd.md
│   │   ├── validate-stories.md
│   │   ├── validate-ac.md
│   │   ├── jira-cleanup.md
│   │   ├── cert-checklist.md
│   │   ├── estimate-review.md
│   │   ├── status-report.md
│   │   ├── meeting-helper.md
│   │   └── gap-analysis.md
│   ├── sub/                           # TBA + shared sub-agents (6 + shared utilities)
│   │   ├── context-loader.md
│   │   ├── catalog-match.md
│   │   ├── jira-read.md
│   │   ├── jira-write.md
│   │   ├── confluence-read.md
│   │   └── confluence-write.md
│   └── skills/                        # 23+ PM skills (organized by domain)
│       ├── discovery/
│       ├── storytelling/
│       ├── tracking/
│       ├── reporting/
│       ├── team-ops/
│       └── admin/
├── agents/                            # 4 orchestrators (from accedo-pm-template)
│   ├── ba-orchestrator.md
│   ├── discovery-agent.md
│   ├── intake-orchestrator.md
│   └── project-kickoff-agent.md
└── docs/
    ├── INTEGRATION_GUIDE.md           # Routing logic + duplication handling
    ├── WORKFLOW_GUIDE.md              # Common workflows (discovery, storytelling, reporting)
    └── REFERENCE.md                   # Quick command + skill lookup
```

---

## Usage Patterns

### Setup (Once per Engagement)

```
/sow-importer → /onboard-project → /gap-analysis
```
Outputs: `project-context.md` (catalog + your choices)

### Feature Work (Repeating)

```
/feature-discovery [feature] → /generate-stories → /validate-stories
```
For bulk generation:
```
/spec-to-tickets [spec] → /ticket-generation
```

### Periodic

```
/jira-cleanup [weekly]
/status-report [format: delivery/sprint/weekly/exec]
/backlog-grooming [sprint]
```

### Full Discovery

```
/discovery-agent [raw SOW/RFP]
```
Chains: sow-importer → onboard-project → gap-analysis → feature-discovery → generate-stories

---

## Team

- **Mike** — TBA agent architecture, OTT catalog, 13 commands + 6 sub-agents
- **Vanessa** — PM skills library (23+), orchestrators, integration + routing
- **Accedo** — OTT domain knowledge, delivery patterns

---

## Customization

**Add new commands** — create `.claude/commands/your-command.md`, link in CLAUDE.md
**Add new skills** — create `.claude/skills/domain/your-skill.md`, reference in docs
**Extend catalog** — edit `OTT-requirements-reference.md` to add systems/features/effort ranges
**Change routing** — edit `CLAUDE.md → INTEGRATION ROUTING` section

---

## Versioning

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-10 | Mike's TBA (13 commands, 6 sub-agents) + Vanessa's 23 skills + 4 orchestrators |

---

## See Also

- `docs/INTEGRATION_GUIDE.md` — detailed routing + duplication handling
- `CLAUDE.md` — agent behavior, output style, typical workflow
- `OTT-requirements-reference.md` — catalog of all OTT capabilities + effort ranges

