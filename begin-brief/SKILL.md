---
name: begin-brief
description: Load full context for a brief, then plan its implementation before writing any code. Use when the user says "begin work on <brief>", "kick off brief <brief>", "start <brief>", or otherwise wants an agent to start working a brief under .agents/briefs/. Reads the brief, its depends_on closure, linked ADRs/PRD sections/specs, and only the relevant docs/style guide(s).
---

# Begin a brief

Load everything needed to work a brief with full grounding, then present a plan and **wait for approval** before writing code. See `CONTEXT.md` for the ubiquitous language and `AGENTS.md` for project conventions.

The cardinal rule: **read the context before you touch code.** A brief is a vertical slice written in the project's ubiquitous language (`CONTEXT.md`) and built on a stack of prior decisions (ADRs), a parent PRD, and the briefs it `depends_on`. Skip any of that and you produce work that ignores resolved decisions — or describes it in words the project has retired.

## Step 1 — Walk the links to assemble the reading set

Start from the brief the user named — accept a path (`.agents/briefs/foo.md`) or a bare ID (`foo` → `briefs/foo.md`). Follow its links with Read, Grep, and Glob, tracking what you have already visited so you never re-read or loop:

1. **Read the starting brief.**
2. **`depends_on` — recurse fully.** Its frontmatter `depends_on:` lists other brief IDs — the substrate this slice lands on. Add each, read it, and repeat for _its_ `depends_on`, until no new IDs appear. Skip any ID already visited; that guards against cycles. A `depends_on` ID with no matching file is **unresolved** — note it, don't crash.
3. **`[[wiki-links]]` — one hop only.** Body links like `[[brief-id]]` are cross-references: read the named brief once for context, but do **not** follow _its_ wiki-links onward — many point at not-yet-built work. A link that resolves to no file (e.g. the literal `[[brief-id]]`) is a syntax example; skip it.
4. **Parent refs — collect and dedupe.** From every brief now in the set, gather the references in its `Parent` section. One pass finds them all:
   ```
   grep -rhoE 'ADR-[0-9]{4}|docs/(prds|specs)/[A-Za-z0-9._/-]+' <brief-files> | sort -u
   ```
   Resolve each `ADR-NNNN` to its file with `ls docs/adrs/NNNN-*.md`; keep the cited `§` section for each `docs/prds/…` reference.

You now hold the full set: the brief, its `depends_on` closure, the one-hop wiki-links, and the ADRs / PRD sections / specs they cite.

## Step 2 — Read it, in priority order

1. **`CONTEXT.md` — first, always.** The ubiquitous language every brief, ADR, and PRD is written in — brief, workspace, worktree, Sandbox, agent, Host, project — plus the workspace `status` enum and the synonyms the project deliberately avoids ("task", "repo", "session", "branch"). It's ~60 lines and the cheapest read in the set; reading it first makes everything below legible and keeps your plan's vocabulary correct. Never skip it.
2. **The brief** — fully. `What to build` + every acceptance criterion + the `Parent` section.
3. **`depends_on` closure** — read each brief fully. If the set is large (more than ~8 briefs), deep-read the starting brief and its **direct** deps; for distant deps, read only `What to build` + acceptance criteria.
4. **One-hop `[[wiki-links]]`** — read for cross-reference context only; don't recurse (Step 1).
5. **ADRs** — read every one cited. They are tiny and they are binding decisions; honor them.
6. **PRD sections / specs** — read the cited `§` sections of `docs/prds/v1.md` and any `docs/specs/*` spec.
7. **Style guide(s)** — read **only** the guide(s) the work touches, and **only the relevant sections**. Pick by what the brief actually builds, not a keyword skim. The guides are the largest single cost, so be deliberate.

## Step 3 — Plan, then wait

Do **not** start coding. Present:

- **Understanding** — 2–4 sentences on what this slice delivers and the prior decisions (ADRs) it must honor.
- **Acceptance criteria** — restated as a checklist; this is the definition of done.
- **Plan** — the deep modules to introduce, where they live, and the test strategy the criteria imply (unit + integration; property-based where edge cases dominate).
- **Recommended approach** — name how you intend to build it and why. When the acceptance criteria are test-driven — deep modules with table-driven or property-based tests, an integration test pinning the slice — recommend a red-green-refactor loop via the `/tdd` skill. For a thin or exploratory slice, recommend building directly. State the recommendation; the user decides.
- **Open questions** — anything ambiguous or any apparent conflict between the brief and an ADR. Surface conflicts; do not silently resolve them.

After planning:

1. Write the plan to `.scratch/plans/<brief-slug>.md`. Resolve naming conflicts by appending a `-<number>` to the filename. `<brief-slug>` is the brief's ID — the `.agents/briefs/<slug>.md` basename.
2. Stop and wait for the user to approve or adjust. If adjustments are requests, update the plan or ask the user clarifying questions.
3. After getting final approval from the user, update the plan if necessary.

## Step 4 — Create the worktree, then build

Once the plan is finalized, isolate the work in its own Git worktree before writing any code. Create it on a fresh branch named for the brief's slug, under the project's `.worktrees/` directory:

```
git worktree add .worktrees/<brief-slug> -b <brief-slug>
```

`.worktrees/` is gitignored and anchored to the project root, so the worktree stays out of the main checkout. If you're resuming a brief whose branch already exists, drop `-b` and point the worktree at that branch instead.

With the worktree in place and work about to start, set the brief's `status:` frontmatter to `in_progress`. Then work the slice from inside that worktree, building it the way the user agreed to and committing incrementally with Conventional Commits (`AGENTS.md`).

## Notes

- If the brief isn't tagged `agent-ready`, say so before planning; it may need triage first (`/triage`).
