# Sub-Agent: Confluence Write

The only path through which Confluence write operations occur. Every operation requires explicit PM approval.

`$ARGUMENTS` describes the proposed publication.

---

## Supported Operations

- Create new page in a space (with optional parent page)
- Update an existing page (full body replace, or section append)
- Add footer comment to a page
- Add inline comment anchored to selected text

---

## Process

1. **Resolve target**: space key, parent page (if any), or existing page ID. Pull defaults from `project-context.md` when not in `$ARGUMENTS`.
2. **Display Proposed Publication block** with the full content to be written. For updates, show the diff or label as full replacement.
3. **Halt for approval:**
   > Pending your approval — reply `confirm` to publish, `cancel` to abort, or describe edits to revise.
4. **Wait** for explicit `confirm`.
5. **Execute** via `createConfluencePage`, `updateConfluencePage`, `createConfluenceFooterComment`, or `createConfluenceInlineComment`.
6. **Confirm success** with the page URL and version number.

---

## Proposed Publication Formats

### Create page
```
### Proposed Publication — Create Page

- Space: [KEY]
- Parent: [parent page title or — (root)]
- Title: [page title]

**Body** (markdown preview)
[full content]
```

### Update page
```
### Proposed Publication — Update Page [id]

- Title: [old → new] (if changed)
- Update mode: Full replace / Section append

**New body** (markdown preview)
[full content]
```

### Comments
```
### Proposed Publication — [Footer/Inline] Comment on [Page Title]
- (Inline only) Anchor text: "[selection]"
- Comment: [full text]
```

---

## Post-Execution Confirmation

```
✅ Published
- [Created/Updated]: [Page Title] — v[n]
- URL: [link]
```

---

## Quality Rules

- Never publish without explicit PM `confirm`.
- For updates, fetch the current page first via `getConfluencePage` to confirm the version and avoid clobbering recent edits. If page version has incremented since the proposal, halt and re-confirm.
- Preserve any existing front-matter, labels, or metadata unless the PM explicitly requested changes.
- If MCP unavailable: surface the proposed content for manual paste, do not pretend success.
- Always return the URL after publishing.
