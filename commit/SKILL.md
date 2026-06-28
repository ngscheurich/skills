---
name: commit
description: Write and create Git commits in house style — Conventional Commits plus the repo's prose conventions. Use when committing changes, drafting or amending a commit message, or whenever the user asks you to commit.
---

# Commit messages

Write every commit as a **Conventional Commit** that also follows the repo's prose conventions. Shape:

```
type(scope): summary

Optional body explaining why, hard-wrapped at 72 columns.

Co-Authored-By: <agent name and version> <noreply address>
```

## Subject line

- `type(scope): summary` — include the scope when the change is local to one area.
- Imperative mood ("add", "drop", "correct"), lowercase after the colon, no trailing period.
- Keep it concise and under 72 characters.
- Describe the change and its intent, not the files touched.

## Types

Pick the type that names the change's intent, not the one matching the largest part of the diff.

- `feat` — a new user-facing capability
- `fix` — a bug fix
- `docs` — documentation only
- `style` — formatting/whitespace, no change in meaning or behavior
- `refactor` — a code change that neither adds a feature nor fixes a bug
- `perf` — a change that improves performance
- `test` — adding or correcting tests
- `build` — build system, tooling, or dependencies
- `ci` — continuous integration config or pipelines
- `chore` — maintenance that touches neither src nor tests
- `revert` — reverts a previous commit

## Body

Include a body whenever the change isn't self-evident from the subject. It explains the **why** — the reasoning, the constraint, the tradeoff — not the what the diff already shows.

- Separate it from the subject with one blank line, and hard-wrap at 72 columns.
- Reference the decision record or acceptance criteria that motivated the change (e.g. `ADR-0011`, `AC #4`) when one applies.
- Skip the body for genuinely trivial commits (a one-line chore, a typo fix).

## Breaking changes

Mark a breaking change with `!` before the colon (`feat(server)!: ...`) and add a `BREAKING CHANGE:` footer describing the break.

## Prose conventions

A commit message is prose — follow [`prose.md`](../prose.md). Read it for the full guidance; the two rules that bite most often in commit messages:

- **Hyphenation:** don't hyphenate settled multi-word terms ("Unix domain socket", "coding agent").
- **Inclusive language:** avoid the terms in the prose guide's table (e.g. "sanity check", "man-in-the-middle") and use its listed alternatives.

## Attribution

End every commit you author with a `Co-Authored-By` trailer, after a blank line, that identifies you — the authoring coding agent — by name and a noreply address. Fill in your own identity:

```
Co-Authored-By: <agent name and version> <noreply address>
```

## Examples

```
feat(briefs): parse the frontmatter map for reads

Parses only the small frontmatter map (scalars and flat lists) via
the frontmatter parser, leaving the body opaque per ADR-0011. Absent fields read as
absent rather than erroring, so partial briefs don't crash reads.

Co-Authored-By: <agent name and version> <noreply address>
```

```
chore: clean up skills lockfile

Co-Authored-By: <agent name and version> <noreply address>
```
