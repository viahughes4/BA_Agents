# Skills Registry — All 54 Accedo PM Skills

**How to invoke:** `/skill-name` or in CLAUDE code: `invoke('/skill-name')`

All skills below are available in `.claude/skills/` and routable via project context.

---

## Discovery (3)

| Skill | Path | Purpose |
|-------|------|---------|
| discovery | `skills/discovery/discovery.md` | Feature intake + cross-domain analysis |
| feature-discovery | `skills/discovery/feature-discovery.md` | 15-dimension feature breakdown (integrates with TBA `/feature-discovery`) |
| gap-analysis | `skills/discovery/gap-analysis.md` | Requirements audit vs. catalog (integrates with TBA `/gap-analysis`) |
| new-feature | `skills/discovery/new-feature.md` | New feature template + intake form |

---

## Storytelling (5)

| Skill | Path | Purpose |
|-------|------|---------|
| generate-stories | `skills/storytelling/generate-stories.md` | Batch story generation (integrates with TBA `/generate-stories`) |
| spec-to-tickets | `skills/storytelling/spec-to-tickets.md` | Spec → structured ticket breakdown |
| generate-prd | `skills/storytelling/generate-prd.md` | Extended PRD assembly (integrates with TBA `/generate-prd`) |
| validate-stories | `skills/storytelling/validate-stories.md` | Extended validation + external checks (integrates with TBA `/validate-stories`) |
| validate-ac | `skills/storytelling/validate-ac.md` | AC quality check (integrates with TBA `/validate-ac`) |

---

## Tracking (5)

| Skill | Path | Purpose |
|-------|------|---------|
| weekly-tracker | `skills/tracking/weekly-tracker.md` | Weekly status + blockers tracker |
| backlog-grooming | `skills/tracking/backlog-grooming.md` | Backlog health + refinement |
| backlog-health | `skills/tracking/backlog-health.md` | Backlog analysis dashboard |
| jira-cleanup | `skills/tracking/jira-cleanup.md` | Broader Jira hygiene (integrates with TBA `/jira-cleanup`) |
| ticket-generation | `skills/tracking/ticket-generation.md` | Batch Jira ticket creation + linking |

---

## Reporting (6)

| Skill | Path | Purpose |
|-------|------|---------|
| status-report | `skills/reporting/status-report.md` | Multi-format status (integrates with TBA `/status-report`) |
| project-health-report | `skills/reporting/project-health-report.md` | Health dashboard + velocity trends |
| release-prep | `skills/reporting/release-prep.md` | Release readiness checklist |
| release-notes | `skills/reporting/release-notes.md` | Release notes generator |
| release-day-runbook | `skills/reporting/release-day-runbook.md` | Release day playbook + comms |
| qa-report | `skills/reporting/qa-report.md` | QA status + test coverage report |

---

## Team-Ops (5)

| Skill | Path | Purpose |
|-------|------|---------|
| standup-prep | `skills/team-ops/standup-prep.md` | Daily standup briefing |
| async-standup | `skills/team-ops/async-standup.md` | Async standup template + digester |
| morning-briefing | `skills/team-ops/morning-briefing.md` | Morning digest (15-min read) |
| meeting-helper | `skills/team-ops/meeting-helper.md` | Meeting notes → actions (integrates with TBA `/meeting-helper`) |
| end-of-day | `skills/team-ops/end-of-day.md` | EOD summary + next-day prep |

---

## Admin (3)

| Skill | Path | Purpose |
|-------|------|---------|
| jira-qa-duplicates | `skills/admin/jira-qa-duplicates.md` | Find + merge duplicate tickets |
| jira-qa-contradictions | `skills/admin/jira-qa-contradictions.md` | Find contradictory stories/specs |
| escalation-brief | `skills/admin/escalation-brief.md` | Escalation digest + mitigation |

---

## Planning (4)

| Skill | Path | Purpose |
|-------|------|---------|
| sprint-planning | `skills/planning/sprint-planning.md` | Sprint intake + capacity planning |
| sprint-gate | `skills/planning/sprint-gate.md` | Sprint readiness gate (spec checks) |
| sprint-closeout | `skills/planning/sprint-closeout.md` | Sprint retro + metrics capture |
| sprint-velocity | `skills/planning/sprint-velocity.md` | Velocity trending + capacity forecast |

---

## Research (23)

| Skill | Path | Purpose |
|-------|------|---------|
| ask | `skills/research/ask.md` | Open question routing (ask Claude anything) |
| biweekly-sync-prep | `skills/research/biweekly-sync-prep.md` | Biweekly sync briefing + agenda |
| cert-checklist | `skills/research/cert-checklist.md` | Platform certification prep (OTT-specific) |
| competitive-analysis | `skills/research/competitive-analysis.md` | Competitive feature audit |
| confluence-email-digest | `skills/research/confluence-email-digest.md` | Confluence → email summary |
| confluence-lookup | `skills/research/confluence-lookup.md` | Confluence search + page reader |
| decision-log | `skills/research/decision-log.md` | Decision tracker + rationale logger |
| dependency-radar | `skills/research/dependency-radar.md` | Cross-team dependency scanner |
| escalation-tracker | `skills/research/escalation-tracker.md` | Active escalation tracker + SLA monitor |
| estimate-review | `skills/research/estimate-review.md` | Estimate review vs. catalog (integrates with TBA `/estimate-review`) |
| mitigation-plan | `skills/research/mitigation-plan.md` | Risk mitigation planner |
| onboarding | `skills/research/onboarding.md` | Team onboarding checklist |
| po-sync-prep | `skills/research/po-sync-prep.md` | PO sync agenda + briefing |
| project-schedule | `skills/research/project-schedule.md` | Project schedule builder + tracker |
| risk-register | `skills/research/risk-register.md` | Risk register + tracking |
| roadmap-sync | `skills/research/roadmap-sync.md` | Roadmap alignment + comms |
| sow-importer | `skills/research/sow-importer.md` | SOW → context extractor |
| status-meeting-prep | `skills/research/status-meeting-prep.md` | Status meeting agenda + materials |
| status-report-email-writer | `skills/research/status-report-email-writer.md` | Email version of status report |
| sync-general-skills | `skills/research/sync-general-skills.md` | General sync + alignment check |
| thread-digest | `skills/research/thread-digest.md` | Slack/email thread → action digest |
| escalation-tracker | `skills/research/escalation-tracker.md` | Track active escalations + SLAs |

---

## Routing by Use Case

### When you need to...

**Discovery**
- `feature-discovery` — analyze a feature request deeply
- `discovery` — general feature intake
- `new-feature` — onboard new feature template

**Storytelling**
- `generate-stories` — write Jira-ready stories
- `spec-to-tickets` — break spec into ticket structure
- `validate-stories` — QA stories for quality + buildability
- `generate-prd` — create client-facing PRD

**Tracking & Hygiene**
- `weekly-tracker` — log weekly status + blockers
- `backlog-grooming` — refine backlog for sprint
- `jira-cleanup` — fix board hygiene issues
- `ticket-generation` — batch-create tickets in Jira
- `jira-qa-duplicates` — find + merge dupes
- `jira-qa-contradictions` — find conflicting specs

**Reporting & Communication**
- `status-report` — create multi-format status
- `project-health-report` — health dashboard
- `release-prep` — pre-release checklist
- `release-notes` — generate release notes
- `release-day-runbook` — release day playbook
- `qa-report` — QA status + metrics

**Team Sync**
- `standup-prep` — daily standup briefing
- `async-standup` — async standup template
- `morning-briefing` — morning digest
- `meeting-helper` — meeting → actions
- `end-of-day` — EOD summary

**Planning**
- `sprint-planning` — sprint intake + capacity
- `sprint-gate` — spec readiness gate
- `sprint-closeout` — sprint retro + velocity
- `sprint-velocity` — velocity trends

**Research & Analysis**
- `gap-analysis` — requirements audit
- `competitive-analysis` — competitor feature audit
- `dependency-radar` — cross-team deps
- `decision-log` — track decisions
- `risk-register` — track risks
- `escalation-tracker` — track escalations
- `estimate-review` — estimate validation
- `mitigation-plan` — risk mitigation

**Admin**
- `escalation-brief` — escalation summary
- `confluence-lookup` — Confluence search
- `confluence-email-digest` — Confluence → email
- `po-sync-prep` — PO sync prep
- `biweekly-sync-prep` — biweekly sync prep
- `status-meeting-prep` — meeting materials
- `thread-digest` — Slack/email digest
- `project-schedule` — schedule + milestones
- `roadmap-sync` — roadmap alignment
- `onboarding` — team onboarding
- `cert-checklist` — platform cert prep
- `ask` — ask Claude anything

---

## Integration with TBA Commands

Skills that **integrate with** TBA commands (they compose rather than duplicate):

| TBA Command | Invokes Skill | How |
|------------|--------------|-----|
| `/feature-discovery` | `feature-discovery` | Calls skill for extended analysis |
| `/generate-stories` | `generate-stories` + `spec-to-tickets` | Skill provides batch generation; command wraps with self-validation |
| `/validate-stories` | `validate-stories` | Skill provides extended validation; command provides 5-lens OTT checks |
| `/generate-prd` | `generate-prd` | Skill generates full PRD; command coordinates scope input |
| `/jira-cleanup` | `jira-cleanup` | Skill handles full cleanup; command is command-specific workflow |
| `/gap-analysis` | `gap-analysis` | Skill provides general gap audit; command provides OTT catalog scoring |
| `/status-report` | `status-report` | Skill provides multi-format output; command coordinates TBA context |
| `/meeting-helper` | `meeting-helper` | Skill extracts actions; command provides TBA-specific signal routing |
| `/estimate-review` | `estimate-review` | Skill compares vs. benchmarks; command provides OTT catalog context |
| `/validate-ac` | `validate-ac` | Skill validates ACs; command is alias/wrapper |

---

## How to Invoke

**Direct invocation (in Claude Code):**
```
/weekly-tracker
/spec-to-tickets [spec]
/sprint-planning [sprint]
```

**From within a skill/command:**
```
Invoke `/weekly-tracker` to log blockers.
Call `/spec-to-tickets` to break this down into tickets.
```

**Programmatically (in agent code):**
```
invoke('/skill-name')
invoke('/skill-name [input]')
```

---

## Discovering the Right Skill

**By workflow phase:**
1. Discovery → `feature-discovery`, `discovery`, `gap-analysis`
2. Storytelling → `generate-stories`, `spec-to-tickets`, `validate-stories`
3. Tracking → `weekly-tracker`, `backlog-grooming`, `jira-cleanup`
4. Planning → `sprint-planning`, `sprint-gate`, `sprint-closeout`
5. Reporting → `status-report`, `project-health-report`, `release-prep`
6. Team-ops → `standup-prep`, `morning-briefing`, `end-of-day`

**By problem:**
- "Stories aren't clear" → `/validate-stories`
- "Backlog is a mess" → `/backlog-grooming` or `/jira-cleanup`
- "Need a status update" → `/status-report` or `/morning-briefing`
- "What's left before release?" → `/release-prep` or `/project-health-report`
- "Team's blocked" → `/escalation-brief` or `/escalation-tracker`
- "Need to know velocity" → `/sprint-velocity`

---

## Availability

All 54 skills are **available in BA_Agents** and ready to invoke from day one. No setup required beyond cloning the repo and running `claude`.

