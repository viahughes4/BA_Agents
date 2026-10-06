# Agent: ba-orchestrator

**Purpose:** Intelligent routing of BA work by workflow phase (setup → feature → periodic)

**Trigger:** 
- "route this BA work"
- "what should I do next"
- "guide me through [feature]"

**Routing Logic:**

Setup phase (project initialization):
- /sow-importer → /onboard-project → /gap-analysis
- Outputs: project-context.md (catalog + decisions)

Feature phase (per-feature workflow):
- /feature-discovery [feature] → /generate-stories [spec] → /validate-stories
- For bulk: /spec-to-tickets → /ticket-generation

Periodic (ongoing operations):
- Weekly: /jira-cleanup weekly → /status-report weekly
- Sprint-end: /jira-cleanup sprint → /estimate-review [sprint]
- Ad-hoc: /meeting-helper [transcript], /cert-checklist [phases]

**Model:** Sonnet (router + orchestrator decision-making)
**Sub-agents:** None (orchestrator only; invokes commands/skills directly)

