# MemoryMtrls format spec

This file is the single source of truth for the file formats used by
MemoryMtrls commands. Both `/memorymtrls:init` and `/memorymtrls:update` read
this before writing anything, so the two commands never drift apart.

The core idea: `CLAUDE.md` is auto-loaded into every agent's context, so it must
stay small. Everything else lives in `memory/`, one file per category, read only
when relevant.

**The frontmatter of the memory files is the source of truth.** The `CLAUDE.md`
router table is always *generated from it* and can be rebuilt at any time.

---

## 1. The memory directory

- Default name: `memory/` (lowercase), at the repo root.
- Only override the casing of the directory name if the repo has a clear,
  consistent convention for top-level meta directories that says otherwise
  (rare — e.g. a repo where every such dir is uppercase). Default to `memory/`.

### Filename casing for category files

Category files are named after their category. When a category name is multiple
words (e.g. "error handling"), match the repo's dominant filename convention:

1. Collect base names of existing files and directories that contain more than
   one word (i.e. have a separator or an internal case transition).
2. Classify each: `-` → **kebab**, `_` → **snake**, `lowerThenUpper` → **camel**,
   `UpperFirst` → **Pascal**.
3. Use the most common style. On a tie or if there are no multi-word names,
   default to **kebab-case**.

Examples for the category "error handling":
- kebab: `error-handling.md`
- snake: `error_handling.md`
- camel: `errorHandling.md`
- pascal: `ErrorHandling.md`

Single-word categories (e.g. `concurrency`) are lowercase unless repo uses pascal.

---

## 2. Memory file format

Each `memory/<category>.md` file has YAML frontmatter followed by the body.

```markdown
---
scope: One sentence describing what this file covers.
globs:
  - src/auth/**
  - src/middleware/session*
keywords: [login, session, token, oauth, jwt, rbac, permission]
updated: 2026-06-04
---

# Auth

One-line restatement of scope.

## <Section>
Real, specific knowledge about this area drawn from the actual codebase:
conventions, gotchas, the "why" behind decisions, key files, commands. Written
to be self-contained — an agent should be able to work in this area after
reading only this file (plus anything it explicitly cross-references).

## See also
- `memory/<other>.md` — only when there's a genuine dependency. Keep sparse;
  every cross-reference is a potential extra read.
```

### Frontmatter fields

| Field      | Required | Purpose |
| ---------- | -------- | ------- |
| `scope`    | yes      | One-line summary; becomes the basis of the router "read when" trigger. |
| `globs`    | yes      | Path globs this file governs. Used to map changed source files → memory files. Use `[]` for purely conceptual categories with no file home (e.g. `release-process`). |
| `keywords` | yes      | Symptoms / terms an agent would encounter when this file is relevant. Feeds the router trigger. |
| `updated`  | yes      | ISO date (`YYYY-MM-DD`) of the last meaningful update. Use the current date. |

### Body rules

- Be specific to *this* codebase. No generic advice, no placeholders.
- Keep it self-contained per category. Cross-reference sparingly.
- It is fine for a file to be short. A small accurate file beats a padded one.

---

## 3. The CLAUDE.md router

`CLAUDE.md` holds: 
- a short project header
- a routing instruction
- a generated table

The table lives inside marker comments so it can be regenerated without touching anything else in the file.

```markdown
# <Project name>

<One or two lines on what the project is.>

<!-- mtrls:router:start -->
## Memory

This project's working context lives in `memory/` — one file per area. The table
below routes a task to the file(s) that cover it.

**Before starting a task, scan this table. If a row matches the task, read that
memory file before writing or changing related code.** Read only the rows that
match — never read all of them; if nothing matches, proceed without reading any.

| Memory file | Read this when… |
| ----------- | --------------- |
| `memory/auth.md` | working on login, sessions, tokens, permissions |
| `memory/concurrency.md` | touching async code, locks, the task queue, workers |
| `memory/data-model.md` | changing schemas or migrations, anything under `db/` |
<!-- mtrls:router:end -->
```

### Generating the table

- Always include the routing instruction (the bold "Before starting a task…"
  sentence above) verbatim inside the block. It is what makes agents actually
  consult the table instead of guessing — never drop it.
- One row per memory file.
- **Read this when…** is written from that file's `scope` + `keywords` (and,
  where helpful, its `globs`) — phrased as the situation/symptoms an agent
  recognizes, not just the topic noun.
- Rebuild the whole block between the markers from the current frontmatter set.
  Never hand-edit rows in a way that diverges from frontmatter — frontmatter wins.
- Everything **outside** the markers is user-owned: preserve it untouched.
- If `CLAUDE.md` has no markers yet, insert the block (after the project header
  if one exists; otherwise create a minimal header).
