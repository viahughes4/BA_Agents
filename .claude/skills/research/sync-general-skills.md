---
name: sync-general-skills
description: Packages all general (non-client-specific) skills, sub-routines, and CLAUDE.md rules from this repo into a single portable markdown file that can be copied to another Claude Code project. Triggers on "sync skills to other project", "export general skills", "create starter pack", "share skills", "portable skills", or any request to bring improvements from this repo into another project.
---

# Skill: Sync General Skills

Produce a portable markdown document containing all general-purpose skills, sub-routines, and CLAUDE.md rules from this repo that are not client-specific. Output is a single file the user can paste into another Claude Code project to bootstrap it with the same patterns.

---

## What is "general" vs "client-specific"

**General (include):** Any skill, rule, or sub-routine that works for any OTT/PM project without modification. These do not reference client folder paths, Jira project keys, Slack channel IDs, Drive file IDs, or client names.

**Client-specific (exclude):** Anything referencing CBC, RDI, cbc-ctv, specific channel IDs (C03CF2H6UHX etc.), specific Drive folder IDs, specific Jira boards, or client contacts.

---

## Process

### Step 1 - Read the general CLAUDE.md rules

Read `CLAUDE.md` in the project root. Extract only the sections that are general rules with no client references:
- Formatting rules (no em dash, etc.)
- Action items tracking rule
- Jira draft-first rule
- Skill-first rule
- HTML export rule
- Weekly tracker auto-sync pattern (without CBC-specific folder paths or Drive IDs)
- Risk register auto-update rule (structural rule only, not the specific file paths)
- Gmail draft-only rule
- Slack draft-to-PM rule

Strip any lines referencing CBC, RDI, specific folder paths, specific channel IDs, specific Drive file IDs, or client names. Generalise them (e.g. replace "CBC" with "[Client]", replace specific folder paths with `[client-folder]/`).

### Step 2 - Identify general skills

These skills are general and should always be included in the export:

| Skill | Notes |
|-------|-------|
| feature-discovery | No changes needed |
| generate-prd | No changes needed |
| generate-stories | No changes needed |
| validate-stories | No changes needed |
| validate-ac | No changes needed |
| gap-analysis | No changes needed |
| risk-register | No changes needed |
| estimate-review | No changes needed |
| ticket-generation | No changes needed |
| spec-to-tickets | No changes needed |
| status-report | No changes needed |
| status-meeting-prep | No changes needed |
| project-health-report | Replace client folder paths with [client] variables |
| standup-prep | Replace client folder paths with [client] variables |
| weekly-tracker | Replace client folder paths with [client] variables |
| sprint-closeout | No changes needed |
| release-prep | No changes needed |
| release-notes | No changes needed |
| release-day-runbook | No changes needed |
| cert-checklist | No changes needed |
| po-sync-prep | Replace client-specific paths |
| meeting-helper | No changes needed |
| mitigation-plan | No changes needed |
| escalation-brief | No changes needed |
| escalation-tracker | No changes needed |
| sow-importer | No changes needed |
| discovery | No changes needed |
| competitive-analysis | No changes needed |
| jira-cleanup | No changes needed |
| jira-qa-contradictions | No changes needed |
| jira-qa-duplicates | No changes needed |
| qa-report | No changes needed |
| end-of-day | Remove CBC-specific Drive IDs and channel IDs, replace with [config] references |
| async-standup | No changes needed |

For each skill, read its SKILL.md file from `.claude/skills/[skill-name]/SKILL.md`.

### Step 3 - Identify general sub-routines

Read and include:
- `.claude/sub/jira-project-snapshot.md` - general, takes project key as input
- `.claude/sub/slack-signal-scan.md` - general, reads from config
- `.claude/sub/slack-context-extractor.md` - general

Exclude: cbc-github-digest.md (hardcoded repo), gmail-cbc-digest.md (CBC-specific), context-loader.md (client paths).

### Step 4 - Check for recent improvements

Read the git log for recent commits affecting .claude/ files:
```bash
git log --oneline --since="30 days ago" -- .claude/
```

Extract any commits that improved general skills or sub-routines this month. Note them in a "Recent improvements" section so the target project knows what has changed.

### Step 5 - Build the portable document

Save to: `exports/[YYYY-MM-DD]-general-skills-export.md`

Create the `exports/` folder if it does not exist.

Structure:

```
# Accedo PM General Skills - Portable Export
**Generated:** [date]
**Source repo:** [repo name]

## How to use this file

1. Copy the CLAUDE.md Rules section into your project's CLAUDE.md
2. Copy each skill file into `.claude/skills/[skill-name]/SKILL.md`
3. Copy each sub-routine into `.claude/sub/[name].md`
4. Create a `.claude/config.yml` with your project's Jira board ID, Slack channels, and client name
5. Update any [client] or [config] placeholders with your project's values

## Recent improvements (last 30 days)
[bullets from git log]

## CLAUDE.md Rules to add
[generalised rules from Step 1]

## General Skills
[For each skill: skill name as a header, then the full SKILL.md content]

## General Sub-routines
[For each sub-routine: name as a header, then the full content]

## Config template
[A blank .claude/config.yml template showing the structure to fill in]
```

### Step 6 - Report to user

Tell the user:
- File saved to `exports/[filename].md`
- How many skills included
- Any skills that needed manual generalisation (had client references)
- Instructions for using it in the other project

---

## Rules

- Never include CBC, RDI, cbc-ctv, channel IDs, Drive folder IDs, or Jira board IDs in the export
- Replace all client-specific values with `[client]`, `[client-folder]`, or `[config]` placeholders
- If a skill cannot be cleanly generalised, note it as "manual review needed" and include it with the client references highlighted
- The output file should be self-contained - someone should be able to use it with no other context
