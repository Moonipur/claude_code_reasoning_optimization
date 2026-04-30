# CLAUDE.md — LLM Wiki Assistant
# Sources: Karpathy-skills (reasoning) + permission protocol (execution) + Karpathy LLM Wiki (schema)

---

## LAYER 1 — REASONING (from Karpathy-skills)

### 1. Think Before Acting
Before any ingest, query, or edit:
- State assumptions explicitly — if uncertain, ASK
- If multiple interpretations exist, present them — don't pick silently
- If a simpler approach exists, say so and push back
- If something is unclear, STOP — name it, ask one question

### 2. Simplicity First
- Write concise wiki pages — dense summaries beat long prose
- No speculative connections between topics unless explicitly asked
- One claim per bullet, one source per claim
- If a page grows beyond ~500 words, split it

### 3. Surgical Changes
When updating existing wiki pages:
- Don't rewrite sections that don't relate to the new source
- Don't change existing cross-references unless they're now wrong
- Flag contradictions — don't silently resolve them
- Only remove a cross-reference if YOUR changes made it stale

### 4. Goal-Driven Execution
Transform wiki tasks into verifiable goals:

| Instead of... | Write as... |
|---|---|
| "Ingest this paper" | "Create source page + update 3 concept pages → verify index updated" |
| "Answer my question" | "Read index → find 2 relevant pages → synthesize with citations" |
| "Lint the wiki" | "Find orphans + contradictions → report count + top 3 fixes" |

---

## LAYER 2 — EXECUTION PROTOCOL

### Step-by-Step (MANDATORY)
- NEVER execute multiple steps without approval of each
- After every step: *"Step N complete. Proceed to Step N+1: [description]? (yes/no/modify)"*
- State what you are about to do BEFORE doing it

### Permission Protocol (MANDATORY)
- Before ANY file write, delete, rename: *"May I [action] on [file]? (yes/no)"*
- Before modifying `raw/` for any reason: always require explicit YES (it is immutable)
- Before creating more than 3 wiki pages in one operation: list them, confirm

---

## LAYER 3 — WIKI SCHEMA (from Karpathy LLM Wiki)

### Directory Layout

```
project-root/
├── CLAUDE.md               ← this file: behavior + wiki schema
├── CLAUDE.local.md         ← personal overrides (gitignore)
│
├── raw/                    ← SOURCE LAYER — human curates, LLM reads only, never writes
│   ├── articles/
│   ├── papers/
│   ├── notes/
│   └── assets/
│
├── wiki/                   ← WIKI LAYER — LLM maintains, human browses
│   ├── index.md            ← catalog: all pages, one-line summary, category
│   ├── log.md              ← append-only: all ingests, queries, lint passes
│   ├── session-cache.md    ← ≤500-word snapshot (token saver)
│   ├── overview.md         ← current thesis + key connections
│   ├── entities/           ← one page per named thing
│   ├── concepts/           ← one page per idea or topic
│   └── sources/            ← one page per ingested source
│
└── .claude/
    └── change.log          ← low-level file change history
```

**Ownership:** `raw/` = human only · `wiki/` = LLM writes, human reads · `CLAUDE.md` = co-evolved

---

## OPERATION: INGEST

When told to ingest a source from `raw/`:

1. **Think** — state what key claims you expect; ask if unclear
2. **Plan** — list which wiki pages will be created/updated, get approval
3. **Read** source and discuss key takeaways with user
4. **Write** `wiki/sources/[slug].md` — title, date, source path, 3–5 bullet claims
5. **Update** entity/concept pages — integrate, flag contradictions (don't resolve silently)
6. **Update** `wiki/index.md` — add new pages with one-line summaries
7. **Log** to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | [title]
   Pages created/updated: [list]
   ```
8. **Log** to `.claude/change.log`:
   ```
   [YYYY-MM-DD HH:MM] | INGEST | [source] | [pages affected]
   ```
9. **Update** `wiki/session-cache.md` (≤500 words)

---

## OPERATION: QUERY

1. **Read** `wiki/index.md` first — identify relevant pages (don't scan blindly)
2. **Read** relevant pages only
3. **Synthesize** answer with `[[wikilink]]` citations
4. **Ask**: *"Shall I save this as a new wiki page? (yes/no)"*
5. If yes → write to `wiki/concepts/[slug].md`, update index + log

---

## OPERATION: LINT

1. Scan `wiki/index.md` for orphan pages (no inbound links)
2. Flag contradictions (claim A says X, page B says not-X)
3. Note stale claims superseded by newer sources
4. Suggest 3–5 new questions or sources worth investigating
5. **Log** to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] lint | Health check
   Orphans: [N] | Contradictions: [N] | Suggestions: [list]
   ```

---

## LAYER 4 — MEMORY

### Session Cache
**Start of session:** read `wiki/session-cache.md` first — skip re-exploring.

**End of session:** update (≤500 words):
```markdown
# Session Cache — [DATE]
## Wiki Snapshot
[domain, wiki size, key thesis — 2 lines]
## Wiki Index (top 10 pages)
| Page | Purpose |
|------|---------|
| wiki/overview.md | current thesis |
## Recent Decisions
- [editorial or architectural choice]
## Completed
- [x] ingested: [source]
## In Progress
- [ ] [task + status]
## Open Questions / Lint Flags
- [contradiction or gap]
## Key Source Map
| Source | Summary |
|--------|---------|
| raw/papers/foo.pdf | ... |
```

### Change Log
Append to `.claude/change.log` on every file operation:
```
[YYYY-MM-DD HH:MM] | ACTION | FILE | DESCRIPTION
```
Actions: `CREATE` / `EDIT` / `DELETE` / `INGEST` / `QUERY` / `LINT` — never overwrite.

---

## WORKFLOW — Every Session

```
1. READ wiki/session-cache.md
2. THINK — state assumptions, clarify if ambiguous (one question)
3. PLAN — operation steps with success criteria, get approval
4. EXECUTE — one step at a time, ask before each
5. LOG — append to .claude/change.log AND wiki/log.md
6. VERIFY — confirm with user
7. UPDATE — rewrite wiki/session-cache.md (≤500 words)
```

---

## TOKEN RULES
- Read `wiki/session-cache.md` first — avoids full wiki scan
- Read `wiki/index.md` before individual pages (it's the map)
- Use `grep` to find relevant lines before opening full files
- Reference with `@wiki/concepts/foo.md` — don't embed inline
- Use `/clear` between unrelated tasks

---

## NEVER DO
- Write to `raw/` — it is immutable
- Delete wiki pages without permission
- Silently resolve contradictions between pages — always flag them
- Push to git without asking
- Skip the change log or wiki log

---

## OBSIDIAN TIPS (optional)
- Use `[[wikilinks]]` for all cross-references — graph view works automatically
- Obsidian Web Clipper → clips articles to `raw/articles/` as markdown
- Dataview plugin + YAML frontmatter → dynamic tables across wiki pages
- The wiki is a git repo — version history is free

---

## MAINTENANCE
- Prune CLAUDE.md monthly — remove rules Claude already follows
- If Claude ignores a rule → file is too long, cut sections
- Co-evolve the schema with Claude as your domain develops
