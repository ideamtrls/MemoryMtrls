# MemoryMtrls

Keep agent context focused by replacing `CLAUDE.md` with a memory system
that loads context only when it is relevant for the task. Keep your context
windows clean and your agents will do better work. What're you waiting for?!

## Why

`CLAUDE.md` is auto-loaded into every agent's context on every task. If that
file becomes big, every agent is loaded up with context that isn't relevant to
the work it is about to do. Most of it, in base cases.

From a different angle, projects have been gathering enormous amounts of
context based on every plan and task list used to get work done. Attempting
to understand the past adds context about what used to be to context about
what is, unnecessarily using more context.

This plugin changes the memory system into something that only describes the
current state of the project. If history is relevant, it is contained in the
description of what is.

MemoryMtrls splits that knowledge into one small file per area under `memory/`,
and turns `CLAUDE.md` into a tiny **router**: a generated table that tells the
agent which memory file to read *when*. The agent reads just the files relevant
to what it's working on.

```
CLAUDE.md  (small, always loaded)
  └─ router table ──► memory/auth.md         (read only when working on auth)
                      memory/concurrency.md  (read only when touching async)
                      memory/data-model.md   (read only when changing schemas)
```

## How it works

- Each `memory/<category>.md` has YAML frontmatter (`scope`, `globs`, `keywords`,
  `updated`) and a body with the project knowledge for that area.
- The frontmatter is the source of truth. The router table in `CLAUDE.md` is
  generated from it and can be rebuilt any time.
- Anything in `CLAUDE.md` outside the `<!-- mtrls:router:* -->` markers is left
  alone.

See [`plugin/SPEC.md`](plugin/SPEC.md) for the file formats.

## Commands

- `/memorymtrls:init` — sets up the system. It reads the repo, suggests
  categories, waits for your approval, then writes the `memory/` files and
  replaces `CLAUDE.md` with a router. It won't write anything until you approve
  the categories.
- `/memorymtrls:update` — run it at the end of a session. It finds what changed,
  updates the affected memory files, and regenerates the router.

## Install

```
/plugin marketplace add ideamtrls/MemoryMtrls
/plugin install memorymtrls@ideamtrls
```

## Usage

1. Run `/memorymtrls:init` and approve the categories.
2. Work as normal. Agents read only the memory files they need.
3. Run `/memorymtrls:update` at the end of a session to keep memory current.
