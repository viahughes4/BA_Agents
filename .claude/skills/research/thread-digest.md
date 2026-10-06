---
name: thread-digest
description: Takes a pasted Slack thread and produces whatever the PM needs from it - a draft Slack message, summary, action items, decision options, open questions, or a Jira ticket draft. Uses project context so names and roles are understood correctly. Triggers on "digest this thread", "summarize this", "draft a message from this thread", "what are the actions from this", "what decisions came out of this", "pull the open questions", "write a message to [person] summarizing this", or any request where the user pastes a Slack conversation and needs something useful from it.
---

# Skill: Thread Digest

Read a pasted Slack thread and produce whatever the PM needs from it. Uses project context to understand who people are and what the project background is.

---

## Process

### 1. Load context

Invoke `.claude/sub/context-loader.md` to load project context, team roster, and client contacts. This lets you resolve first names and handles correctly ("Andy" = Andy Bartkiv, Accedo Engineering Lead, etc.).

If context-loader is unavailable, proceed and note that names may be unresolved.

### 2. Accept the thread

The PM pastes the Slack thread directly. Accept it in any format: raw copy-paste, exported JSON, or manually typed summary. Do not ask for a specific format.

If no thread is pasted yet, ask:
> "Paste the Slack thread and tell me what you need from it."

### 3. Parse the thread

Extract the following from the thread, regardless of what output the PM wants:

**Participants:** Who sent messages. Resolve to full names and roles using project context where possible.

**Timeline:** When the conversation happened (if timestamps visible).

**What happened:** A factual 2-3 sentence summary of the conversation - what was being discussed, what triggered it, what the outcome was.

**Decisions made:** Any explicit agreements, sign-offs, or confirmed directions. Look for "let's go with", "agreed", "confirmed", "we'll do", "yes that's right", "sounds good" followed by a direction.

**Action items:** Anything assigned to a specific person. Look for "can you", "I'll", "you should", "let's get", "follow up", names followed by tasks.

**Open questions:** Anything raised but not answered. Look for "?", "not sure", "TBD", "need to confirm", "check with", unanswered asks.

**Blockers:** Anything preventing forward progress - waiting on someone, access issues, technical unknowns.

**Key context:** Any project-specific references (ticket numbers, systems, platform names) that give the PM useful context without them having to re-read the full thread.

### 4. Ask what the PM needs

After parsing, present a brief 2-3 line summary of what you found, then ask:

> "Got it. What do you need from this?
> - **Slack draft** - message to send someone (tell me who and the tone)
> - **Summary** - paragraph of what happened
> - **Actions** - list of action items with owners
> - **Decisions** - what was agreed vs still open
> - **Open questions** - list of unresolved items
> - **Ticket** - Jira draft if this needs to be tracked
> - **All of the above**"

If the PM already stated what they want in their original message (e.g. "draft a message to Harsh"), skip this step and go directly to producing that output.

### 5. Produce the requested output(s)

#### Slack draft
A message the PM can send directly. Ask if not already clear:
- Who is it going to?
- What tone? (professional update / casual check-in / action request / decision ask)

Format:
```
Hi [name],

[Opening - context on why you're reaching out]

[Body - the relevant summary, decision, or ask. 2-4 short paragraphs or bullets depending on complexity.]

[Closing - what you need from them, if anything, and by when]

[Sign-off]
```

Rules for Slack drafts:
- No em dashes - use hyphens or colons instead
- Match the PM's usual communication style - direct, professional, not overly formal
- Never invent facts not in the thread
- If the PM is asking for a decision or action from the recipient, make that the last clear line

#### Summary
A single paragraph, 3-6 sentences. What happened, what was decided, what's still open. Plain English - no jargon. Someone who wasn't on the thread should be able to read this and understand the situation.

#### Action items
```
| Owner | Action | Source | By When |
|-------|--------|--------|---------|
| [Name] | [What they need to do] | [From thread] | [Date if stated, else TBD] |
```

Only include actions that are clearly assigned. If an action is implied but not explicitly assigned, flag it as "Unassigned" and note it should be confirmed.

#### Decisions
```
**Decided:**
- [Decision] - confirmed by [who] on [date if known]

**Still open:**
- [Item] - decision needed from [who]
- [Item] - needs more information before deciding
```

#### Open questions
A numbered list. For each: what the question is, who raised it, and who needs to answer it (if clear from the thread).

#### Ticket draft
Follow the `/ticket-generation` skill format. Determine the ticket type from the thread:
- Is this a bug? Use Bug format.
- Is this a task or investigation? Use Task format.
- Is this an external dependency / follow-up? Use Improvement format.

Always show the draft and get explicit PM approval before offering to create in Jira.

#### All of the above
Produce all sections in one output, separated by clear headers.

---

## Rules

- **Never fabricate.** Every output must trace to something in the thread. If you don't know who owns an action, say "unassigned - confirm with team."
- **Use project context.** Resolve names, understand ticket references, know which platforms and systems are relevant.
- **Slack drafts are for PM review.** Never send directly - always present the draft and let the PM decide to send it.
- **Tickets are drafts first.** Never create in Jira without showing the draft and getting explicit approval.
- **No em dashes** anywhere in the output.
- **If the thread is ambiguous**, state the ambiguity rather than guessing. "It's unclear whether Andy or Eduardo owns this follow-up - confirm before proceeding."
- **Keep outputs scannable.** Bullets over paragraphs wherever possible. The PM should be able to read the output in 60 seconds.

---

> **Skill verification:** Confirm thread-digest was run, parsed the thread, and produced at least one output. If not, run from the top.
