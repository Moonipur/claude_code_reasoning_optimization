# CLAUDE.md — Optimized Claude Code Configuration
# Token-efficient · High accuracy · Step-by-step · Permission-gated

---

## 🧠 CORE BEHAVIOR RULES

### Step-by-Step Execution (MANDATORY)
- **NEVER** execute multiple steps at once without explicit approval of each step
- After each step, STOP and ask: *"Step N complete. Proceed to Step N+1: [description]? (yes/no/modify)"*
- If uncertain at any point, ASK — never assume intent
- State what you are about to do BEFORE doing it

### Permission Protocol (MANDATORY)
- Before ANY file write, delete, rename, or overwrite → ask: *"May I [action] on [file]? (yes/no)"*
- Before running shell commands → show the exact command and ask for confirmation
- Before installing packages → list them and confirm
- Before touching config files (`.env`, `package.json`, etc.) → always require explicit YES

### No Assumptions Policy
- If a requirement is ambiguous → stop and ask a single clarifying question
- Do not infer file paths, variable names, or logic from context alone
- When multiple approaches exist → present options, wait for choice

---

## 📋 CHANGE LOG (Auto-append on every change)

**Every modification MUST be logged to `.claude/change.log` in this format:**

```
[YYYY-MM-DD HH:MM] | ACTION | FILE | DESCRIPTION
```

**Example entries:**
```
[2025-04-30 10:22] | CREATE  | src/utils/parser.ts     | Added CSV parser utility
[2025-04-30 10:25] | EDIT    | src/index.ts            | Imported parser, wired to route
[2025-04-30 10:30] | DELETE  | src/utils/old-parser.ts | Removed deprecated parser
[2025-04-30 10:35] | RUN     | npm install csv-parse    | Added csv-parse dependency
```

Create `.claude/change.log` if it does not exist. Never overwrite existing log entries — always append.

---

## 🗂️ SESSION CACHE — 500-Word Summary

**At the END of every session (or when asked), write/update `.claude/session-cache.md`:**

The summary must be ≤500 words and include:
1. **Project snapshot** — tech stack, key folders, entry points
2. **Recent decisions** — architectural or design choices made
3. **Completed tasks** — what was built/fixed/changed
4. **In-progress tasks** — what was started but not finished
5. **Blockers / open questions** — anything unresolved
6. **Key file map** — 5–10 most important files and their purpose

**At the START of every session**, read `.claude/session-cache.md` first (if it exists).
This replaces the need to re-explore the codebase and saves hundreds of tokens per session.

Template:
```markdown
# Session Cache — Last updated: [DATE]
## Project Snapshot
[tech stack, main folders, entry points — 3–4 lines]

## Recent Decisions
- [decision 1]
- [decision 2]

## Completed
- [x] task 1
- [x] task 2

## In Progress
- [ ] task A (status: ...)

## Blockers / Open Questions
- [question or issue]

## Key File Map
| File | Purpose |
|------|---------|
| src/index.ts | App entry point |
| src/routes/ | All API routes |
```

---

## ⚡ TOKEN OPTIMIZATION RULES

### Read Efficiently
- Read only the files needed for the current task — do NOT scan the whole codebase
- Use `grep` or `find` to locate relevant lines before reading full files
- When referencing docs, use `@docs/filename.md` — do NOT embed full file content inline
- Read the session cache first; skip re-exploration if already covered

### Write Efficiently
- Provide diffs or targeted edits — avoid rewriting entire files unless necessary
- Use short, precise code comments — no verbose inline explanations
- Avoid duplicating logic; reference existing helpers instead

### Context Control
- Use `/clear` between unrelated tasks to reset context
- Avoid `/compact` unless absolutely necessary (it's slow and opaque)
- Keep this CLAUDE.md file under 200 lines — prune rules Claude already follows naturally

---

## 🎯 WORKFLOW — Standard Task Execution

Follow this sequence for every task:

```
1. READ session cache (.claude/session-cache.md)
2. CLARIFY — ask one question if anything is ambiguous
3. PLAN — write out steps in numbered list, get approval
4. EXECUTE — one step at a time, ask before each
5. LOG — append each change to .claude/change.log
6. VERIFY — run tests or confirm output with user
7. UPDATE cache — rewrite .claude/session-cache.md (≤500 words)
```

---

## 🔧 CODE STYLE (Adjust to your project)

- Prefer existing patterns in the codebase over introducing new ones
- Follow the language's idiomatic style (e.g., PEP 8 for Python, ESLint rules for JS/TS)
- Use descriptive names — avoid single-letter variables outside of loops
- Write one test per behavior, not one test per function

---

## 🚫 NEVER DO

- Never delete files without explicit permission
- Never push to git without asking
- Never run `rm -rf` or destructive commands without confirmation
- Never expose secrets, API keys, or credentials in output
- Never skip the change log
- Never assume a task is complete without user confirmation

---

## 📁 FILE LAYOUT FOR CLAUDE MEMORY

```
project-root/
├── CLAUDE.md               ← This file (shared, checked into git)
├── CLAUDE.local.md         ← Personal overrides (add to .gitignore)
└── .claude/
    ├── change.log          ← Auto-appended on every change
    └── session-cache.md    ← ≤500 word project summary, updated each session
```

---

## 🔄 MAINTENANCE

- Review and prune CLAUDE.md monthly — remove rules Claude already follows
- If Claude ignores a rule → the file may be too long; cut unused sections
- If Claude asks questions already answered here → reword that section more clearly
- Test changes by observing Claude's behavior, not by reading the file
