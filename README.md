# claude-md-templates

> Two `CLAUDE.md` templates for Claude Code — built from three sources: Karpathy's coding principles, a strict execution protocol, and Karpathy's LLM Wiki pattern.

---

## Templates

| File | Use when... |
|------|------------|
| [`CLAUDE-coding.md`](#claude-codingmd) | You want Claude Code to be a disciplined coding assistant |
| [`CLAUDE-wiki.md`](#claude-wikimd) | You want Claude Code to build and maintain a personal knowledge wiki |

---

## Architecture — Three Layers

Both templates are built from the same three-layer stack, stacked in this order:

```
┌─────────────────────────────────────────────┐
│  LAYER 1 — REASONING                        │  ← Karpathy-skills repo
│  Think · Simplify · Surgical · Goal-Driven  │    (how Claude reasons)
├─────────────────────────────────────────────┤
│  LAYER 2 — EXECUTION PROTOCOL               │  ← this repo's original work
│  Step-by-step · Permission · No assumptions │    (how Claude acts)
├─────────────────────────────────────────────┤
│  LAYER 3 — MEMORY                           │  ← this repo's original work
│  Change log · Session cache (≤500 words)    │    (how Claude remembers)
└─────────────────────────────────────────────┘
```

`CLAUDE-wiki.md` adds a fourth layer on top:

```
┌─────────────────────────────────────────────┐
│  LAYER 4 — WIKI SCHEMA                      │  ← Karpathy LLM Wiki gist
│  Ingest · Query · Lint · raw/ + wiki/       │    (how Claude builds knowledge)
└─────────────────────────────────────────────┘
```

---

## Credits & Sources

| Source | What it contributes |
|--------|-------------------|
| [Karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) | Four reasoning principles derived from Karpathy's observations on LLM coding pitfalls |
| [Karpathy LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | Three-layer knowledge base architecture (raw → wiki → schema) + ingest/query/lint operations |
| This repo | Execution protocol (step-by-step, permission-gated), change log, session cache |

---

## `CLAUDE-coding.md`

A lean coding assistant config. Drop it as `CLAUDE.md` in any code project.

### What each layer does

**Reasoning (Layer 1)** — Before Claude writes a single line, it must:
- State assumptions explicitly and ask rather than guess
- Propose the simplest solution that solves the problem
- Touch only what the task requires — nothing adjacent
- Define verifiable success criteria before starting

**Execution (Layer 2)** — How Claude runs each task:
- Stops after every step and waits for your approval
- Asks permission before any file write, delete, or shell command
- Never infers — always asks one clarifying question when ambiguous

**Memory (Layer 3)** — How Claude carries context across sessions:
- Appends every file change to `.claude/change.log` (timestamped, append-only)
- Reads `.claude/session-cache.md` at the start of each session instead of re-scanning your codebase
- Rewrites the session cache (≤500 words) at the end of each session

### Install

```bash
# new project
curl -o CLAUDE.md https://raw.githubusercontent.com/Moonipur/claude_code_reasoning_optimization/main/Reasoning/CLAUDE.md
mkdir -p /your/project/.claude

# existing project (append)
cat CLAUDE-coding.md >> /your/project/CLAUDE.md
```

### File layout

```
project-root/
├── CLAUDE.md           ← drop the template here
├── CLAUDE.local.md     ← personal overrides (gitignore this)
└── .claude/
    ├── change.log      ← auto-appended on every change
    └── session-cache.md← ≤500-word snapshot
```

---

## `CLAUDE-wiki.md`

A knowledge base config. Drop it as `CLAUDE.md` in a research or notes project. Claude becomes the writer and maintainer of a compounding personal wiki — you source content, Claude builds the knowledge base.

### What each layer does

**Reasoning (Layer 1)** — Adapted for wiki work:
- Think before ingesting — state expected key claims, ask if unclear
- Write concise pages — dense summaries over long prose
- Don't silently resolve contradictions — flag them
- Define verifiable goals before each operation (ingest/query/lint)

**Execution (Layer 2)** — Same permission and step-by-step protocol as the coding template, adapted for wiki operations.

**Memory (Layer 3)** — Same change log. Session cache now includes wiki index, thesis, and source map.

**Wiki Schema (Layer 4)** — Three named operations:

| Operation | Trigger | What Claude does |
|-----------|---------|-----------------|
| **Ingest** | "ingest raw/papers/foo.pdf" | Reads source, writes summary page, updates entity/concept pages, updates index + log |
| **Query** | any question | Reads index → finds relevant pages → synthesizes with `[[citations]]` → optionally files answer as new page |
| **Lint** | "lint the wiki" | Finds orphan pages, flags contradictions, suggests new sources |

### Install

```bash
# new wiki project
cp CLAUDE-wiki.md /your/wiki/CLAUDE.md
mkdir -p /your/wiki/{raw/{articles,papers,notes,assets},wiki/{entities,concepts,sources},.claude}

# then bootstrap
cd /your/wiki && claude
# tell Claude: "bootstrap this wiki — create wiki/index.md, wiki/log.md, wiki/session-cache.md"
```

### Add your first source

```bash
# paste or clip an article
echo "# Article Title\n..." > raw/articles/my-article.md

# tell Claude Code:
# "ingest raw/articles/my-article.md"
```

Claude will walk through the ingest workflow step-by-step, asking approval before touching each file.

### File layout

```
project-root/
├── CLAUDE.md               ← drop the template here
├── CLAUDE.local.md         ← personal overrides (gitignore)
│
├── raw/                    ← your sources (immutable — LLM never writes here)
│   ├── articles/
│   ├── papers/
│   ├── notes/
│   └── assets/
│
├── wiki/                   ← LLM-maintained knowledge base
│   ├── index.md            ← catalog of all pages
│   ├── log.md              ← append-only operation history
│   ├── session-cache.md    ← ≤500-word snapshot
│   ├── overview.md         ← current thesis
│   ├── entities/
│   ├── concepts/
│   └── sources/
│
└── .claude/
    └── change.log
```

### Pair with Obsidian (optional)

Open the project root as an Obsidian vault. The wiki uses `[[wikilinks]]` natively — graph view, backlinks, and Dataview queries all work without extra setup.

---

## Design Principles

**Keep CLAUDE.md short.** LLMs can reliably follow ~150–200 instructions total, and Claude Code's own system prompt already uses ~50 of those slots. `CLAUDE-coding.md` is 149 lines; `CLAUDE-wiki.md` is 212. Both stay well within the effective range.

**Only include what applies every session.** Task-specific knowledge lives in `wiki/` or `docs/` and is referenced on demand with `@wiki/concepts/foo.md` — not embedded in `CLAUDE.md` on every session.

**The session cache is the biggest token saver.** Instead of Claude scanning dozens of files at the start of each session, it reads one ≤500-word summary. This single habit reduces per-session token overhead significantly on any non-trivial project.

**The Karpathy reasoning principles fill a gap.** Our execution protocol (step-by-step, permission-gated, logged) governs *how Claude acts*. The Karpathy principles govern *how Claude thinks before acting*. Neither replaces the other.

**Treat CLAUDE.md like code.** Review it when Claude gets things wrong. Prune rules Claude already follows. Test changes by observing behavior, not by reading the file.

---

## License

MIT — copy, fork, adapt.
