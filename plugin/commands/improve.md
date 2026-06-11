---
description: Audit and improve the memory system — enforce SPEC standards, rewrite history-speak into present state, verify accuracy, and regenerate the router.
---

You are auditing and improving the MemoryMtrls memory system. Agents update
memory files continuously as they work; this command is the periodic quality
pass that keeps those updates honest. It takes no arguments and can be run at
any time — it judges the memory files as they are now, not what happened in
this session.

First, read `${CLAUDE_PLUGIN_ROOT}/SPEC.md` for the standards you are
enforcing, then the repo's `CLAUDE.md` router, then every memory file.

If there is no memory directory yet, tell the user to run `/memorymtrls:init`
first, and stop.

## Step 1 — Inventory

List the memory files and read them all. For a small set, run the audits below
inline, file by file. For a large set, fan out **parallel audit agents** (one
per memory file), each applying Steps 2–5 to its file and returning a list of
findings + fixes; tell the user how many you're dispatching first.

## Step 2 — Standards audit

Check each file against SPEC.md and fix what's off:
- Frontmatter has exactly `scope`, `globs`, `keywords`, `updated`, with valid
  values (globs that match real paths, a real ISO date, keywords that reflect
  the body).
- Filenames follow the repo's casing convention.
- Body follows the SPEC shape: a title, a one-line restatement of scope,
  sections of real knowledge, sparse `See also`.

## Step 3 — Tense audit (what is, never what was)

This is the heart of the command. Memory files describe the **current state**
of the system; they are not changelogs. Hunt for history-speak — "previously",
"used to", "no longer", "was removed", "renamed from", "migrated to", "now
uses", "instead of the old…" — and rewrite each occurrence as if the current
state had always been true:
- If the *reasoning* behind a past decision still matters, keep it as a
  present-tense fact ("sessions live in Redis because the app runs on multiple
  hosts"), not as an event.
- Content describing removed features is deleted outright — unless the
  *absence* is itself a gotcha, in which case describe the absence in the
  present tense.

## Step 4 — Accuracy audit

A subtly wrong memory file is worse than none. Check the content against the
**current** source tree:
- Spot-check each file's key claims in the actual code; fix what's stale.
- Verify build/test/run commands still exist as written.
- Check that the `globs` across all files still partition the source tree:
  flag gaps (source with no memory home) and overlaps.
- Cross-references between memory files resolve.

## Step 5 — Quality improvements

Improve where the standard is met but the content is weak — keeping every edit
surgical:
- Deduplicate: a fact should have one home; replace duplicates with a
  cross-reference only if the dependency is genuine.
- Trim padding and generic advice that isn't specific to this codebase. A
  small accurate file beats a padded one.
- Tighten `scope` and `keywords` where they've drifted from the body.

Bump `updated` only on files whose body you actually changed.

## Step 6 — Taxonomy check (propose only)

If the audits reveal a structural problem — two files that should merge, one
that should split, a subsystem with no memory home — **propose** the change
(names, scopes, "read when" triggers) and wait for the user's approval. The
user owns the taxonomy; never reorganize it unilaterally.

## Step 7 — Regenerate the router

Rebuild the `<!-- memorymtrls:start -->` … `<!-- memorymtrls:end -->` block in
`CLAUDE.md` from the current frontmatter set (per SPEC.md §4), including both
bold instructions verbatim. Leave everything outside the markers untouched.

## Step 8 — Report

Summarize per file: what the audits found and what you fixed (standards, tense
rewrites, accuracy corrections, quality trims). Call out anything left for the
user to decide. If everything was already clean, say so plainly.
