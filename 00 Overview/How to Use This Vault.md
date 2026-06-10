---
tags: [overview, meta]
---

# How to Use This Vault

## Opening it
Obsidian → **Open folder as vault** → select the `KnowGap Vault` folder. That's it — every note is a plain Markdown file.

## Obsidian features this vault leans on
- **Wikilinks** `[[Like This]]` — click to jump; hover (Ctrl/Cmd) to preview. Links are the whole point: follow them instead of reading folders linearly.
- **Graph view** (Ctrl/Cmd+G) — because notes link along real architectural lines, the graph roughly *is* the architecture. Try local graph (right-click a note) on [[System Architecture]].
- **Backlinks panel** — open `MongoDB Notes` and see every note that references it.
- **Tags** — `#flow`, `#concept`, `#reference`, `#team`, plus per-repo `#frontend` / `#backend` / `#extension`. Click a tag to list all matching notes.
- **Mermaid diagrams** render natively (used in architecture and flow notes).
- **Checkboxes** in [[Onboarding Checklist]] and [[Open Questions]] are clickable.

## Conventions (keep these when adding notes)
1. **Layered detail:** overview notes link *down* to file guides and flows; never bury big-picture info in a detail note.
2. **One topic per note**, named so the `[[link]]` reads naturally in a sentence.
3. **Frontmatter:** `tags`, plus `repo:`/`file:` when a note is about specific code.
4. **Flows over files:** when documenting a new feature, write a `Flow - ...` note first (sequence of calls across repos) — it ages better than screenshots.
5. **Trust the code:** when a note disagrees with the repo, fix the note and (if it's interesting) log the discrepancy in [[Open Questions]].
6. Use the [[Templates/Code Note Template|templates]] for new code/meeting notes.

## Sharing with the team
Best option: **make this folder a git repo** and push it alongside the project repos — markdown diffs cleanly, and everyone opens the same folder in their own Obsidian.
```bash
cd "KnowGap Vault" && git init && git add -A && git commit -m "Project knowledge vault"
```
Add `.obsidian/workspace*` to `.gitignore` (per-user UI state); committing the rest of `.obsidian/` is optional (shares settings/plugins).

## Keeping it alive
- After fixing a confusing bug → add 3 lines to the relevant note.
- After a meeting → new note in `06 Team/` from the meeting template; link decisions to affected notes.
- New feature → new `Flow -` note + update the file guides it touches.
- A vault only stays useful if updating it is *cheaper than re-figuring things out* — keep notes short and linked rather than long and complete.
