---
name: tour
description: Author a comprehensive, top-to-bottom developer tour of the whole project or one subsystem, built from the source — terminology first, broad strokes then every rabbit hole, verbatim code labelled with file:line, and Mermaid diagrams for processes. Use when the user asks to deeply explain, document, or onboard someone to the architecture of the project or a subsystem from the code itself. Not for narrating a range of commits — that is the `recap` skill.
---

# Tour

Produce one long-form Markdown document that guides a developer through a subject end to end: what it is, the terminology, the shape, then every load-bearing detail, showing real code and diagramming real flows. The output is durable onboarding documentation built from the source, not a narration of recent changes.

## Scope it first

Decide what the tour covers before writing:

- **Whole project** — the orientation doc a new contributor reads once. Runs big-picture → cross-cutting foundations → each subsystem → appendices.
- **One subsystem** — a focused deep-dive (the transport layer, the agent runtime). Same rules, narrower blast radius.

Name the file for the scope: `.agents/tours/<scope>.md` (e.g. `.agents/gtours/project.md`, `docs/tours/agent-runtime.md`). Resolve conflicts by appending `-<number>`.

## Steps

1. **(Large scopes) Work in isolation.** For a whole-project tour, update the trunk and create a Worktree/branch under `.worktrees/`, then `cd` into it and confirm with `git branch --show-current` before writing. Skip for a small subsystem doc.
2. **Ground in the canonical sources first.** Read the domain/architecture anchors before writing a word, so the document inherits the project's own vocabulary and structure: the domain-model doc, the contributor/agent guide, the relevant entrypoints, and the decision records (ADRs). Read any existing tour as a _style_ reference, not content to copy.
3. **Fan out, then synthesize.** A whole-project tour is too much for one pass — dispatch parallel subagents, each owning one slice (a server area, a client area, the decision-record set, specs, build tooling), each returning verbatim excerpts **with `file:line` labels** and the load-bearing facts. Then write the document in one narrative voice; do not staple reports together.
4. **Write to the house rules** (below).
5. **Land it.** Commit as `docs:` with an explicit pathspec and a body that summarizes the document and names any code/doc divergences it flagged. Open a PR only when asked.

## Writing rules

- **Terminology first.** Open with the ubiquitous language so later sections use precise terms. Pull them from the domain doc; do not invent synonyms.
- **Broad strokes, then rabbit holes.** Each part states the shape, then dives. Err toward over-specificity — the job is to be exhaustive.
- **Show the code.** When discussing code, paste it verbatim in a fenced block labelled `path:line`. Excerpts must match source exactly; quote, then explain around the block. Highlight the interesting hunks, don't dump whole files.
- **Diagram the processes.** For any non-trivial flow, include a Mermaid diagram (sequence, state, or ER as fits). Diagrams illustrate; prose carries detail.
- **Verify divergences empirically.** Where code and docs/ADRs disagree, confirm the truth at the keyboard before writing it, and flag the divergence in the document rather than silently picking one.
- **Respect prose style.** Honor the project style guides — inclusive language, no over-hyphenation, no editor-specific section banners.

## Notes

- This differs from the `recap` skill: that narrates a range of commits; this builds a from-the-source guide to how the system works.
