# Agent: discovery-agent

**Purpose:** 5-phase OTT discovery workflow (raw SOW → execution-ready)

**Trigger:**
- "run discovery"
- "start discovery"
- "/discovery"
- User shares SOW/RFP and expects comprehensive discovery

**5-Phase Workflow:**

1. **Intake** (1-2 hrs)
   - /sow-importer [SOW/RFP doc]
   - /onboard-project [fill gaps]
   → Outputs: project-context.md

2. **Analysis** (2-3 hrs)
   - /gap-analysis [project-context]
   - Identify missing requirements + AI-codegen readiness gaps
   → Outputs: Gap report + flagged items

3. **Feature Discovery** (1-2 hrs per feature)
   - /feature-discovery [each flagged feature]
   - Light mode (trivial) or Full 15D (non-trivial)
   → Outputs: Discovery report per feature

4. **Storytelling** (2-4 hrs)
   - /generate-stories [per feature spec]
   - /validate-stories [self-validation]
   → Outputs: Jira-ready stories

5. **Prep** (1-2 hrs)
   - /cert-checklist [platforms + timeline]
   - /estimate-review [effort vs. catalog]
   → Outputs: Cert timeline + estimate validation

**Parallelization:**
- Phase 1 (intake) and Phase 5 (prep) can run in parallel
- Features in Phase 3 can discover in parallel

**Model:** Opus (complex multi-phase coordination)
**Sub-agents:** None (invokes TBA commands + skills sequentially)

**Output:**
- project-context.md (locked per PM approval)
- Feature discovery reports
- Jira stories (Minimal/Standard/Heavy)
- Cert checklist + estimate validation
- Ready for sprint planning

