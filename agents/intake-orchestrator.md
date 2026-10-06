# Agent: intake-orchestrator

**Purpose:** Route any raw input (Slack, doc, email, meeting note) to structured Jira tickets

**Trigger:**
- "intake this"
- "turn this into tickets"
- "ticket this up"
- "create a ticket for [input]"
- User pastes spec/feature/requirement expecting tickets

**Workflow:**

1. **Parse** (15-30 min)
   - If raw feature: /feature-discovery [input]
   - If spec/SOW: /sow-importer [doc] or /spec-to-tickets [text]

2. **Validate** (15-30 min)
   - /gap-analysis [parsed spec]
   - Flag missing requirements

3. **Generate** (30-60 min)
   - /spec-to-tickets [spec] → structured breakdown
   - /ticket-generation [breakdown] → create Jira tickets

4. **Quality Gate** (15-30 min)
   - /validate-stories [created tickets]
   - Resolve any AI-codegen readiness gaps

**Model:** Sonnet (parsing + routing + validation)
**Sub-agents:** None (orchestrates TBA commands + skills)

**Output:**
- Created Jira tickets (linked, prioritized)
- Validation report (ready for sprint)

