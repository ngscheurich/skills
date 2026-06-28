---
name: to-briefs
description: Break a plan, spec, or PRD into independently-grabbable briefs in the project's filesystem brief Provider, using tracer-bullet vertical slices. Use when the user wants to convert a plan into briefs, author briefs from a PRD, or break work down into briefs.
---

# To briefs

Break a plan into independently-grabbable **briefs** using vertical slices (tracer bullets). A brief is a markdown file with YAML frontmatter that records desired changes to the project so an **agent** can grab it in a workspace. See `CONTEXT.md` for the ubiquitous language.

## brief format (recap)

- Lives at `briefs/<brief-id>.md` at the project root.
- Filename stem is the **brief ID** (kebab-case, stable, unique within the project).
- First H1 of the body is the canonical title — do not duplicate it in frontmatter.
- YAML frontmatter fields:
  - `status:` — one of `draft | ready | in_progress | done | closed` (this skill writes `ready` by default, or `draft` for a slice the author has not committed to land; the system rewrites it on workspace transitions per ADR-0008). `ready` means *accepted, intended to land*; `draft` means *captured but not yet committed*. The status axis (commitment) is orthogonal to the triage tags (specification/disposition) — a brief may be `draft` and `agent-ready` at once.
  - `type:` — e.g. `feature`, `bug`, `refactor`, `chore`, `docs`.
  - `tags:` — flat YAML list. Triage dimension lives here (`agent-ready`, `human-only`, `needs-info`, …) per the v1 PRD's resolved decisions.
  - `depends_on:` — flat YAML list of brief IDs the brief is blocked on; missing/renamed entries render as "unresolved" rather than crash.
- Cross-brief references in the body use wiki-links: `[[brief-id]]`.

## Process

### 1. Gather context

Work from whatever is already in the conversation. If the user passes a path (e.g. a PRD under `docs/prds/`, an existing brief, a spec under `docs/specs/`), read its full body. If they pass a `[[brief-id]]` reference, resolve it to `briefs/<id>.md`.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand current state. brief titles and bodies MUST use the project's ubiquitous language from `CONTEXT.md`, and avoid the synonyms `CONTEXT.md` flags. Respect ADRs in the area you're touching; if you'd contradict one, surface it explicitly rather than silently overriding.

### 3. Draft vertical slices

Break the plan into **tracer bullet** briefs. Each brief is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Tag each slice with its triage disposition:

- **`agent-ready`** — fully specified, can be implemented and shipped by an agent without further human input.
- **`human-only`** — requires a human in the loop (architectural decision, design review, hardware access, judgment call). Prefer `agent-ready` where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (data, lifecycle, CLI/TUI, tests).
- A completed slice is demoable or verifiable on its own.
- Prefer many thin slices over few thick ones.
- Slices should be small enough that a single workspace can ship one without growing unwieldy. </vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name (will become the brief's first H1).
- **Proposed brief ID**: kebab-case filename stem.
- **Triage tag**: `agent-ready` / `human-only`.
- **Type**: `feature` / `bug` / `refactor` / etc.
- **Depends on**: which other slices (if any) must complete first — by proposed brief ID.
- **User stories covered**: which user stories this addresses (if the source material has them).

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the `depends_on` relationships correct?
- Should any slices be merged or split further?
- Are the correct slices tagged `agent-ready` vs `human-only`?
- **Which slices are still drafts** — captured but not yet committed to land? Those get `status: draft`; everything else is born `ready` (accepted). This is a commitment question, separate from whether a slice is well-specified: a slice can be `agent-ready` and still a `draft` if you haven't decided to land it.
- Are the proposed brief IDs good? (they become filenames and `[[wiki-link]]` targets, so they should be stable and memorable)

Iterate until the user approves the breakdown.

### 5. Publish the briefs

For each approved slice, write a file at `briefs/<brief-id>.md` (create `briefs/` if it doesn't exist). Write briefs in dependency order (blockers first) so `depends_on:` entries reference real brief IDs that already exist on disk.

Use this template:

<brief-template>
```markdown
---
status: <ready|draft>
type: <feature|bug|refactor|chore|docs>
tags:
  - <agent-ready|human-only>
  - <any other relevant tag>
depends_on:
  - <blocking-brief-id>
  - <another-blocking-brief-id>
---

# <Title goes here — this is the canonical title>

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Use the ubiquitous language from `CONTEXT.md`.

Avoid specific file paths or code snippets — they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it here and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Parent

A reference to the source material the slice was derived from (e.g. `[[parent-brief-id]]`, `docs/prds/v1.md`, `docs/specs/agent-protocol.md`). Omit if the slice was generated from ad-hoc conversation.
```
</brief-template>

Notes on writing the brief:

- Set `status: ready` for an accepted slice (the default), or `status: draft` for a slice the author flagged as not-yet-committed. The system owns the `status:` field after workspace creation (ADR-0008); the author's job is to mark a new brief as accepted-and-grabbable (`ready`) or as an uncommitted candidate (`draft`). Promoting `draft → ready` later is a deliberate hand-edit (acceptance), not a triage side-effect.
- Omit `depends_on:` entirely (rather than writing `depends_on: []`) when there are no blockers.
- Omit `tags:` entirely when there are no tags — but the triage tag is mandatory, so in practice there is always at least one.
- Do NOT include the title in frontmatter. The first H1 of the body is canonical.
- Reference other briefs in prose with `[[brief-id]]` wiki-links.

Do NOT modify the source material (PRD, spec, parent brief). The skill produces new briefs; it does not edit existing ones.
