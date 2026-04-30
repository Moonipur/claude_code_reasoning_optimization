# CLAUDE.md — Second Brain Root
# Scope: ALL sessions in this vault · keep lean · project rules live in projects/*/CLAUDE.md

---

## FIRST RUN — Bootstrap Checklist

Run this ONCE when setting up the vault. Ask permission before each step.

```
[ ] 1. Create directory structure (see layout below)
[ ] 2. Create wiki/index.md       — empty catalog, two sections: Global | Projects
[ ] 3. Create wiki/log.md         — empty, append-only
[ ] 4. Create wiki/session-cache.md — first snapshot (see format below)
[ ] 5. Create wiki/overview.md    — one paragraph: what this vault is for
[ ] 6. Configure Obsidian         — see Obsidian Setup section below
[ ] 7. Verify .gitignore          — add CLAUDE.local.md, .obsidian/workspace*
```

After bootstrap: tell Claude "bootstrap complete" so it marks all boxes done in log.md.

---

## VAULT LAYOUT

```
second-brain/                   ← Obsidian vault root
├── CLAUDE.md                   ← THIS FILE — always loaded, global rules
├── CLAUDE.local.md             ← personal overrides, gitignored
│
├── wiki/                       ← GLOBAL WIKI — cross-project knowledge
│   ├── index.md                ← two-level catalog (categories + counts only at root)
│   ├── log.md                  ← append-only operation history
│   ├── session-cache.md        ← ≤500-word snapshot, loaded every session
│   ├── overview.md             ← vault thesis + key connections
│   ├── concepts/               ← ideas that span projects (tech, science, mental models)
│   ├── entities/               ← people, tools, orgs referenced across projects
│   └── active/                 ← hot pages being actively developed (frontmatter: status: active)
│
├── projects/                   ← PROJECT WIKIS — one subfolder per project
│   └── [project-name]/
│       ├── CLAUDE.md           ← project schema — lazy-loaded when you cd here
│       ├── wiki/
│       │   ├── index.md        ← full catalog for this project
│       │   ├── session-cache.md← project-specific cache
│       │   ├── concepts/
│       │   ├── entities/
│       │   └── sources/
│       └── raw/                ← immutable sources for this project
│           ├── articles/
│           └── assets/
│
├── personal/                   ← PERSONAL WIKI — journals, goals, health
│   ├── CLAUDE.md               ← personal schema — lazy-loaded
│   ├── wiki/
│   └── raw/
│
└── .claude/
    └── change.log              ← timestamped file change history
```

**Ownership:**
- `raw/` anywhere → human curates, LLM reads only, NEVER writes
- `wiki/` anywhere → LLM writes, human reads and browses
- `projects/*/CLAUDE.md` → domain-specific rules, NOT loaded from vault root

---

## OBSIDIAN SETUP

Configure these once after bootstrap. Tell Claude what you've enabled.

**Core settings:**
- Settings → Files & links → Default location for new notes: `wiki/active`
- Settings → Files & links → Attachment folder: `projects/[name]/raw/assets`
- Settings → Files & links → Use `[[Wikilinks]]`: ON
- Settings → Editor → Strict line breaks: OFF

**Recommended plugins (community):**
- Dataview — queries wiki pages by frontmatter (`status`, `domain`, `tags`)
- Obsidian Web Clipper — clips web articles to `raw/articles/` as markdown
- Git — auto-commits wiki changes (pairs with change.log)
- Templater — page templates for concepts, entities, source summaries

**After clipping an article** with Web Clipper:
- Move clipped file to correct `raw/articles/` folder
- Then tell Claude: `ingest [path]`

---

## WIKI CONVENTIONS (enforced on every write)

### Page size limit: ≤300 words per page
If a page exceeds 300 words → split it. Smaller pages = cheaper reads.

### Atomic claims format
Every bullet = one claim + one source citation. No narrative paragraphs.

```markdown
- Attention scales QK dot products by √d to reduce gradient variance [[sources/attention-paper]]
- RLHF requires a reward model trained on human preference pairs [[sources/instructgpt]]
```

Never: long prose that re-explains the source. Compress to claims only.

### Frontmatter on every wiki page
```yaml
---
title: [page title]
domain: [AI | coding | personal | finance | ...]
status: active | archived
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [[sources/slug]]
---
```

`status: active` = Claude reads this in normal sessions
`status: archived` = invisible to Claude unless explicitly requested (grep only)

### Wikilinks for all cross-references
Always use `[[page-name]]` — never plain text references to other pages.
Cross-project links: `[[../projects/project-a/wiki/concepts/foo]]`

### Two-level index rule
`wiki/index.md` at root = categories + page counts ONLY:
```markdown
## Global wiki index
### concepts/ — 42 pages
### entities/ — 18 pages
### active/ — 6 pages

## Projects
### projects/project-a — 28 pages · [[projects/project-a/wiki/index]]
### projects/project-b — 15 pages · [[projects/project-b/wiki/index]]
```

Full page catalogs live in each project's own `wiki/index.md`.

---

## OPERATIONS

### Ingest
Trigger: `"ingest [raw/path/to/file]"`

```
1. THINK — state expected key claims, flag if unclear
2. PLAN — list pages to create/update, ask permission
3. READ source, discuss takeaways with user
4. WRITE wiki/[project]/sources/[slug].md
   - frontmatter + title + source path + ≤5 atomic claims
5. UPDATE relevant concept/entity pages — flag contradictions, don't resolve silently
6. UPDATE correct wiki/index.md — increment count, add row if new page
7. LOG to wiki/log.md:
   ## [YYYY-MM-DD] ingest | [title]
   Pages: [list]
8. LOG to .claude/change.log
9. UPDATE session-cache.md
```

### Query
Trigger: any question about wiki content

```
1. READ wiki/session-cache.md → wiki/index.md (root, then project if scoped)
2. READ only relevant pages (grep first, don't scan everything)
3. ANSWER with [[wikilink]] citations
4. ASK: "Save this as a wiki page? (yes/no)"
5. If yes → write to concepts/[slug].md, update index + log
```

### Lint
Trigger: `"lint"` or `"health-check the wiki"`

```
1. Scan index for orphan pages (no inbound [[links]])
2. Flag contradictions between pages
3. Flag pages over 300 words → suggest splits
4. Flag archived pages still referenced as active
5. Suggest 3 new questions or sources worth investigating
6. LOG to wiki/log.md:
   ## [YYYY-MM-DD] lint
   Orphans: N | Oversized: N | Contradictions: N
```

---

## SESSION CACHE FORMAT

**Every session: read `wiki/session-cache.md` FIRST** before touching any other file.

**End of session: rewrite it** (≤500 words total):

```markdown
# Session Cache — [YYYY-MM-DD]

## Vault Snapshot
[One sentence: what this vault is for and current size]
Global wiki: [N] concepts, [N] entities, [N] active pages
Projects: [list with page counts]

## Wiki Index (active pages only)
| Page | Domain | Purpose |
|------|--------|---------|
| wiki/concepts/attention.md | AI | how transformer attention works |

## Recent Decisions
- [editorial or architectural choice made]

## Completed This Session
- [x] ingested: [source title]
- [x] updated: [page]

## In Progress
- [ ] [task + status]

## Open Questions / Lint Flags
- [contradiction, orphan, or gap found]

## Hot Sources
| Source | Location | Key claim |
|--------|----------|-----------|
| Attention is All You Need | projects/ai-research/raw/papers/ | scaled dot-product attention |
```

---

## BEHAVIOR RULES (always on)

### Step-by-step (MANDATORY)
- State what you will do BEFORE doing it
- After each step: *"Done. Proceed to [next step]? (yes/no/modify)"*

### Permission (MANDATORY)
- Before any file write/delete/rename: *"May I [action] on [file]?"*
- Before writing to `raw/` for any reason: STOP — it is immutable
- Before creating 3+ pages at once: list them, confirm

### No assumptions
- Ambiguous request → ask one clarifying question, then stop
- Multiple valid approaches → present options, wait for choice

### Surgical changes
- Edit only what the current task requires
- Don't fix adjacent issues without asking
- Flag contradictions — never silently resolve them

---

## CHANGE LOG

Append to `.claude/change.log` on every file operation:
```
[YYYY-MM-DD HH:MM] | ACTION | FILE | DESCRIPTION
```
`ACTION`: CREATE / EDIT / DELETE / INGEST / QUERY / LINT
Never overwrite. Always append. Create file if missing.

---

## TOKEN RULES

- Read `wiki/session-cache.md` first — avoids full vault scan
- Read root `wiki/index.md` for orientation, project index for detail
- `grep` before opening any file — locate relevant lines first
- Reference with `@wiki/concepts/foo.md` — never embed full content inline
- Only read `status: active` pages — archived = grep only
- Use `/clear` between unrelated tasks

---

## NEVER DO
- Write to `raw/` — immutable
- Load project/personal CLAUDE.md from root — they're lazy-loaded automatically
- Delete wiki pages without permission
- Silently resolve contradictions — always flag
- Create pages over 300 words without splitting
- Push to git without asking

---

## MAINTENANCE
- Prune this file monthly — remove rules Claude already follows
- If Claude ignores a rule → file too long, cut sections
- Co-evolve with Claude — add rules when Claude gets things wrong
- Run lint monthly to keep graph healthy
