# CLAUDE.md — Behavior Rules + LLM Wiki Schema
# Based on: Karpathy's LLM Wiki pattern + token-efficient Claude Code config

---

## 🧠 CORE BEHAVIOR RULES

### Step-by-Step Execution (MANDATORY)
- NEVER execute multiple steps at once without explicit approval of each step
- After each step, STOP and ask: *"Step N complete. Proceed to Step N+1: [description]? (yes/no/modify)"*
- State what you are about to do BEFORE doing it

### Permission Protocol (MANDATORY)
- Before ANY file write, delete, rename, or overwrite → ask: *"May I [action] on [file]? (yes/no)"*
- Before running shell commands → show the exact command and ask for confirmation
- Before touching config files → always require explicit YES

### No Assumptions Policy
- If a requirement is ambiguous → stop and ask one clarifying question
- Do not infer file paths, variable names, or logic unless already in the wiki

---

## 📁 DIRECTORY LAYOUT

```
project-root/
├── CLAUDE.md               ← This file: behavior rules + wiki schema
├── CLAUDE.local.md         ← Personal overrides (add to .gitignore)
│
├── raw/                    ← SOURCE LAYER — immutable, never modified by LLM
│   ├── articles/
│   ├── papers/
│   ├── notes/
│   └── assets/             ← Images downloaded locally (Obsidian Web Clipper)
│
├── wiki/                   ← WIKI LAYER — LLM-maintained markdown files
│   ├── index.md            ← Catalog: every page, one-line summary, category
│   ├── log.md              ← Append-only: all ingests, queries, lint passes
│   ├── session-cache.md    ← ≤500-word project snapshot (token saver)
│   ├── overview.md         ← Synthesis: current thesis, key connections
│   ├── entities/           ← One page per named thing (person, project, tool)
│   ├── concepts/           ← One page per idea or topic
│   └── sources/            ← One page per ingested source (summary + key claims)
│
└── .claude/
    └── change.log          ← Timestamped record of every file change
```

**Ownership rules:**
- `raw/` → Human curates. LLM reads only. Never modifies.
- `wiki/` → LLM owns. Human reads and browses. Edits by human are allowed.
- `CLAUDE.md` → Co-evolved by human and LLM over time.

---

## 📥 OPERATION: INGEST

When told to ingest a source from `raw/`:

1. **Clarify** — confirm the source file path before starting
2. **Read** — read the source; discuss key takeaways with the user
3. **Request permission** — *"May I create/update these wiki pages: [list]?"*
4. **Write summary** → `wiki/sources/[slug].md` with:
   - Title, date, source path, 3–5 bullet key claims
5. **Update entity/concept pages** — integrate new info; flag contradictions
6. **Update `wiki/index.md`** — add new pages with one-line summaries
7. **Append to `wiki/log.md`**:
   ```
   ## [YYYY-MM-DD] ingest | [source title]
   Pages created/updated: [list]
   ```
8. **Append to `.claude/change.log`**:
   ```
   [YYYY-MM-DD HH:MM] | INGEST | [source path] | [pages affected]
   ```
9. **Update `wiki/session-cache.md`** (≤500 words, see format below)

---

## 🔍 OPERATION: QUERY

When asked a question against the wiki:

1. **Read `wiki/index.md`** first — identify relevant pages
2. **Read relevant pages** — do NOT read the whole wiki blindly
3. **Synthesize answer** with citations: `[[wiki/concepts/foo]]`
4. **Ask permission to save** — *"Shall I file this answer as a new wiki page? (yes/no)"*
5. If yes → write to `wiki/concepts/[slug].md`, update index + log

---

## 🔧 OPERATION: LINT

When asked to health-check the wiki:

1. Scan `wiki/index.md` for orphan pages (no inbound links)
2. Flag contradictions between pages (claim A says X, claim B says not-X)
3. Note stale claims superseded by newer sources
4. Suggest 3–5 new questions or sources worth investigating
5. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] lint | Health check
   Orphans: [N] | Contradictions: [N] | Suggestions: [list]
   ```

---

## 🗂️ SESSION CACHE FORMAT

**Read `wiki/session-cache.md` at the START of every session** — skip re-exploring if already summarised.

**Write/update at the END of every session** (≤500 words):

```markdown
# Session Cache — Last updated: [DATE]

## Project / Wiki Snapshot
[Domain, tech stack, main wiki size — 2–3 lines]

## Wiki Index (top 10 pages)
| Page | Purpose |
|------|---------|
| wiki/overview.md | Current thesis |
| wiki/entities/X.md | ... |

## Recent Decisions
- [architectural or editorial choice]

## Completed This Session
- [x] ingested: [source]
- [x] updated: [page]

## In Progress
- [ ] [task + status]

## Open Questions / Lint Flags
- [contradiction or gap found]

## Key Source Map
| Source | Summary |
|--------|---------|
| raw/papers/foo.pdf | ... |
```

---

## 📋 CHANGE LOG

**Every file change MUST be appended to `.claude/change.log`:**

```
[YYYY-MM-DD HH:MM] | ACTION | FILE | DESCRIPTION
```

Actions: `CREATE` / `EDIT` / `DELETE` / `INGEST` / `QUERY` / `LINT` / `RUN`

Create the file if it doesn't exist. Never overwrite — always append.

---

## ⚡ TOKEN OPTIMIZATION

- Read `wiki/session-cache.md` first — avoid re-exploring the wiki from scratch
- Read `wiki/index.md` before individual pages (it's the map)
- Use `grep` to locate relevant lines before reading full files
- Reference docs with `@wiki/concepts/foo.md` — do NOT embed full content inline
- Provide targeted edits — avoid rewriting entire pages unless necessary
- Use `/clear` between unrelated tasks

---

## 🚫 NEVER DO

- Never modify anything in `raw/` — it is immutable
- Never delete wiki pages without explicit permission
- Never push to git without asking
- Never skip the change log or wiki log
- Never assume a task is complete without user confirmation

---

## 🔄 WORKFLOW — Standard Task Execution

```
1. READ wiki/session-cache.md (skip re-exploration)
2. CLARIFY — ask one question if anything is ambiguous
3. PLAN — write numbered steps, get approval
4. EXECUTE — one step at a time, ask before each
5. LOG — append to .claude/change.log AND wiki/log.md
6. VERIFY — confirm output or run tests
7. UPDATE — rewrite wiki/session-cache.md (≤500 words)
```

---

## 🌐 OBSIDIAN TIPS (optional but recommended)

- Use `[[wikilinks]]` for all cross-references between wiki pages
- Use Obsidian Web Clipper → saves articles to `raw/articles/` as markdown
- After clipping: hotkey downloads images to `raw/assets/`
- Graph view shows hubs, orphans, and clusters across your wiki
- Dataview plugin: add YAML frontmatter to pages for dynamic tables
- The wiki is a git repo — version history and branching come for free

---

## 🔧 MAINTENANCE

- Review and prune CLAUDE.md monthly — remove rules Claude already follows
- If Claude ignores a rule → file is too long; cut unused sections
- Co-evolve the schema with Claude as you discover what works for your domain
- Add emphasis (IMPORTANT, YOU MUST) to rules that keep getting missed
