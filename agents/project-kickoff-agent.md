# Agent: project-kickoff-agent

**Purpose:** Pre-sales → fully structured, execution-ready project (4-phase kickoff)

**Trigger:**
- "run kickoff"
- "start kickoff"
- "new project"
- "I have a new SOW"
- User shares pre-sales materials expecting complete project setup

**4-Phase Workflow:**

1. **Discovery** (1-2 hrs)
   - /sow-importer [sales materials / SOW]
   - /onboard-project [PM intake]
   → Outputs: project-context.md (locked)

2. **Scoping** (2-3 hrs)
   - /gap-analysis [scope validation]
   - /feature-discovery [major epics/features]
   → Outputs: Scope report + missing decisions

3. **Epics & Tickets** (3-4 hrs)
   - /generate-prd [if client contract requires]
   - /spec-to-tickets [epic-level breakdown]
   - /ticket-generation [create Jira tickets]
   → Outputs: Epic backlog + release plan

4. **Team & Schedule** (2-3 hrs)
   - Team assignment + capacity planning
   - Schedule milestones + cert dates
   - Initial velocity estimate
   → Outputs: Project dashboard (team + timeline)

**Parallelization:**
- Phase 1 (discovery) and Phase 4 (team/schedule) can run in parallel

**Model:** Opus (complex multi-phase project setup)
**Sub-agents:** None (invokes TBA commands + skills sequentially)

**Output:**
- Locked project-context.md
- Scope report + decisions documented
- Epic backlog in Jira (prioritized)
- Release schedule + cert timeline
- Team assignments + velocity baseline
- **Ready for sprint planning**

