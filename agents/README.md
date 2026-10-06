# BA_Agents Orchestrators

Four orchestrators route workflows using TBA commands + PM skills:

## ba-orchestrator
**Intelligent routing by BA workflow phase**

Detects your request and picks the right tool:
- Setup phase: sow-importer → onboard-project → gap-analysis
- Feature phase: feature-discovery → generate-stories → validate-stories
- Periodic: jira-cleanup, status-report (runs skills directly)

**Use:** `"route this BA work"`, `"what should I do next for [feature]"`, `"run BA workflow"`

**Invokes:**
- TBA commands: sow-importer, onboard-project, gap-analysis, feature-discovery, generate-stories, validate-stories
- Skills: jira-cleanup, status-report, backlog-grooming

---

## discovery-agent
**5-phase OTT discovery (raw SOW → execution-ready)**

Orchestrates end-to-end discovery:
1. **Intake:** sow-importer + onboard-project
2. **Analysis:** gap-analysis
3. **Feature Breakdown:** feature-discovery (per feature)
4. **Storytelling:** generate-stories + validate-stories
5. **Prep:** cert-checklist, estimate-review

**Use:** `"run discovery"`, `"start discovery"`, `"/discovery"`

**Best for:** New OTT engagements, SOW parsing, kickoff prep

**Invokes:**
- TBA commands: sow-importer, onboard-project, gap-analysis, feature-discovery, generate-stories, validate-stories, cert-checklist
- Skills: validate-stories (extended), gap-analysis (domain check)

---

## intake-orchestrator
**Raw input → structured Jira tickets**

Routes any input (Slack, doc, email, meeting note) to tickets:
1. **Parse:** sow-importer or feature-discovery
2. **Validate:** gap-analysis
3. **Generate:** spec-to-tickets → ticket-generation
4. **Quality Gate:** validate-stories

**Use:** `"intake this"`, `"turn this into tickets"`, `"ticket this up"`, `"make tickets from this"`

**Best for:** Batch ticket creation, spec-to-Jira pipelines

**Invokes:**
- TBA commands: sow-importer, feature-discovery, gap-analysis, validate-stories
- Skills: spec-to-tickets, ticket-generation, validate-stories

---

## project-kickoff-agent
**Pre-sales → fully structured, execution-ready project**

4-phase orchestration:
1. **Discovery:** sow-importer + onboard-project
2. **Scoping:** gap-analysis + feature-discovery
3. **Epics:** spec-to-tickets (epic-level breakdown)
4. **Schedule + Team:** project-schedule (if available) + team assignment

**Use:** `"run kickoff"`, `"start kickoff"`, `"new project"`, `"I have a new SOW"`

**Best for:** Pre-sales handoff, project setup, scope lock

**Invokes:**
- TBA commands: sow-importer, onboard-project, gap-analysis, feature-discovery, generate-prd
- Skills: spec-to-tickets, ticket-generation, project-health-report (initial baseline)

---

## Decision Tree

```
User input
├─ "What should I do next?"
│  └─ ba-orchestrator (phases + routing)
├─ "Start discovery on this SOW"
│  └─ discovery-agent (5-phase OTT workflow)
├─ "Turn this into Jira tickets"
│  └─ intake-orchestrator (spec → tickets)
├─ "Kickoff this new project"
│  └─ project-kickoff-agent (pre-sales → execution)
└─ "Run this specific command"
   └─ Direct command (e.g., /generate-stories)
```

---

## When to Use Which

| Scenario | Agent | Why |
|----------|-------|-----|
| New OTT engagement from SOW | discovery-agent | End-to-end 5-phase workflow |
| Setup + phased decision-making | ba-orchestrator | Routing by phase (setup → feature → periodic) |
| Bulk tickets from spec | intake-orchestrator | Spec → Jira pipeline |
| Pre-sales handoff + kickoff | project-kickoff-agent | 4-phase kickoff sequence |
| Specific task (1 command) | None—use direct TBA command | Skip orchestration |

---

## Customizing Agents

Each agent is defined in this directory:
- `ba-orchestrator.md` — routing logic + phase detection
- `discovery-agent.md` — 5-phase discovery workflow
- `intake-orchestrator.md` — spec-to-Jira routing
- `project-kickoff-agent.md` — pre-sales to execution

To customize:
1. Edit the agent's `.md` file (instructions + routing rules)
2. Update the decision logic (which commands/skills to invoke)
3. Test the workflow end-to-end

---

## Parallel Execution

Agents can run in parallel when phases are independent:
- discovery-agent Phase 1 (intake) runs while Phase 5 (prep) handles cert + estimates
- intake-orchestrator can parse + validate in parallel

See each agent's `.md` for parallelization rules.

