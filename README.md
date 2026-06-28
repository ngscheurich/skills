# Agent skills

This is my living set of [Agent Skills](https://agentskills.io/home). Many of these originated in [Matt Pocock's collection](https://github.com/mattpocock/skills).

## Skills

- **[begin-brief](begin-brief/SKILL.md)** — Load full context for a brief (its `depends_on` closure, linked ADRs/PRD sections/specs, and relevant docs), then plan its implementation and wait for approval before writing code.
- **[commit](commit/SKILL.md)** — Write Git commits in house style: Conventional Commits plus the repo's prose conventions.
- **[grill-me](grill-me/SKILL.md)** — Interview you relentlessly about a plan or design, one question at a time, until you reach shared understanding.
- **[grill-with-docs](grill-with-docs/SKILL.md)** — A grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates documentation (CONTEXT.md, ADRs) inline as decisions crystallise.
- **[land-brief](land-brief/SKILL.md)** — Clean up locally after a brief branch merges on GitHub: update main, mark the brief done, and tear down branches and worktrees.
- **[multi-review](multi-review/SKILL.md)** — Code-review a vertical slice by running multiple reviewer subagents in parallel against a shared rubric, having a judge rank them, then synthesizing one coalesced review.
- **[recap](recap/SKILL.md)** — Generate a commit-by-commit recap of unpushed work or a chosen commit range, then wait for annotation.
- **[tdd](tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop, favoring behavior-focused integration tests over implementation-coupled ones.
- **[to-briefs](to-briefs/SKILL.md)** — Break a plan, spec, or PRD into independently-grabbable briefs using tracer-bullet vertical slices.
- **[to-prd](to-prd/SKILL.md)** — Turn the current conversation context into a PRD written to `docs/prds/<name>.md`.
- **[tour](tour/SKILL.md)** — Author a comprehensive, top-to-bottom developer tour of the whole project or one subsystem, built from the source — terminology first, broad strokes then every rabbit hole, verbatim code labelled with `file:line`, and Mermaid diagrams for processes.
- **[triage](triage/SKILL.md)** — Move briefs through a small state machine driven by triage tags.
- **[write-a-skill](write-a-skill/SKILL.md)** — Create new agent skills with proper structure, progressive disclosure, and bundled resources.
