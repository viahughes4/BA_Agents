---
name: morning-briefing
description: Use this skill for the daily morning briefing and digest. Triggers on "morning briefing", "daily digest", "what's on my plate today", "what do I need to know today", "action items for today", "catch me up", "brief me", "start my day", or any request for a consolidated morning overview. Orchestrates confluence-email-digest, standup-prep, weekly-tracker, and project-health-report into a single scannable briefing.
---

# Skill: Morning Briefing

This is a skill entry point for the morning-briefing agent. It exists so you can invoke `/morning-briefing` directly.

---

## Process

Read `.claude/agents/morning-briefing.md` and follow its complete process:

1. **Wave 0**: Run shared sub-routines in parallel (gmail-cbc-digest, slack-signal-scan, jira-project-snapshot)
2. **Wave 1**: Invoke skills using pre-loaded data (confluence-email-digest, standup-prep, weekly-tracker in load mode, project-health-report)
3. **Wave 2**: Consolidate all outputs into a single morning briefing
4. **Wave 3**: Update the weekly tracker with new action items from the briefing, run python3 scripts/md-to-html.py on it, sync to Google Drive folder 1ficiL4i0DJTKwpb770YoE8h1RtNEkaHe, and commit.

Follow the agent file exactly. Do not skip waves or omit skills.

---

## Rules

- This skill delegates entirely to the morning-briefing agent. Do not deviate from its process.
- All rules in the agent file apply (wave ordering, pre-set answers, file saves, deduplication, no fabrication).
- After the briefing, offer the follow-up options listed in the agent file.
- After any .md file is saved (standup notes, health report, morning briefing), immediately run python3 scripts/md-to-html.py <path> per repo rules.
- After the tracker is updated in Wave 3, sync it to Google Drive folder 1ficiL4i0DJTKwpb770YoE8h1RtNEkaHe using mcp__claude_ai_Google_Drive__create_file.
- If the agent file is unavailable or a wave fails, skip that wave, note the failure in the briefing output, and continue with remaining waves rather than halting entirely.

---

> **Skill verification:** Please ensure that the morning-briefing agent workflow was actually executed in full. If any wave was skipped, re-run from the beginning.
