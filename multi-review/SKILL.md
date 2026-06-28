---
name: multi-review
description: Thoroughly code-review a vertical slice by running two or more reviewer subagents in parallel against a shared rubric, having a judge rank them, then synthesizing one coalesced review. Reviewer count is configurable. Use when the user wants a deep/competitive/multi-agent code review of a slice, feature, brief, or the current diff — "multi-review this slice", "competitive review", "have N agents review X and merge it".
---

# Multi-Review

Run a competitive, multi-agent code review of one vertical slice. Two or more reviewer subagents review the same slice independently against [rubric.md](rubric.md), a judge ranks them, and you synthesize the field into a single coalesced review written to `.agents/reviews/`.

## 1. Resolve the slice

Accept any of three targets and pin down the file set before dispatching:

- **Current diff / branch** (default if nothing named) — `git diff main...HEAD` plus uncommitted changes. The slice is whatever changed.
- **A brief slug** — read the brief under `.agents/briefs/`, then trace the code that implements it across every layer it touches.
- **A free-text feature / entry files** — trace the named feature top to bottom through each layer it crosses.

A _vertical slice_ crosses layers, so the boundaries matter: include any wire/protocol boundary and the points where data is decoded or validated as it passes between layers. Produce a concrete list of files (with the interesting line ranges) and the entry points; this is the slice definition every reviewer receives.

[slices.md](slices.md) explains how to identify a vertical slice and trace it across the layers it crosses — start there when the user names a feature, and use it to sharpen a diff or brief target into the full set of layers it touches.

## 2. Gather the standards

Collect what the reviewers must hold the code against, so each one reads the same canon:

- The rubric: [rubric.md](rubric.md).
- Style guides for the file types touched: the project's language style guide(s).
- Prose + naming: the project's prose guide (honor its avoid-list).
- Domain truth: `CONTEXT.md`, plus any ADRs/specs the slice's code or brief cite.

Pass these as paths, not pasted contents — reviewers read them themselves.

## 3. Dispatch the reviewers (parallel, competing)

Spawn **N** reviewer subagents, where **N is the count the user passes** (e.g. `/multi-review 4 <target>` → 4 reviewers). Clamp to **2 ≤ N ≤ 6**; default to 2 when no count is given. Launch them in a **single message with N agent calls** so they run concurrently. Give every reviewer the identical brief: the slice definition, the standards paths, the finding format from the rubric, and this framing:

> You are competing against other reviewers on the same slice. A judge will score your review against the rubric. Be exhaustive and precise: every finding cites `file:line`, names its rubric dimension and severity, states the concrete problem, and proposes a fix. Read the code and the cited standards yourself —  do not trust the slice summary alone. Unsupported or speculative findings will be scored against you.

Each reviewer returns a structured review (severity-ordered findings + an overall read), per the rubric's finding format.

## 4. Judge the field

Spawn one **judge** subagent. Give it the slice definition, the rubric, and all reviews verbatim. It must: score each review on the rubric dimensions, name a **winner** (the most thorough _and_ accurate), flag any finding it judges wrong or unsupported, and list the strongest **unique** catches from each non-winning review. The judge ranks; it does not write the final review.

## 5. Synthesize one coalesced review

Build the final review yourself using the winner as the spine: graft in the unique findings the judge credited to the other reviewers, drop anything the judge flagged as wrong, de-dupe overlaps, and re-sort every finding by severity. Attribute nothing to individual reviewers in the body — it reads as one review.

Write it to `.agents/reviews/<slug>.md` (slug = brief name, branch, or a short feature kebab; resolve conflicts with `-<n>`). Open with a one-paragraph verdict and a severity tally, then the findings. End the file with a short `## Provenance` note: the targets, reviewer count, and which review the judge ranked first. Print the verdict and tally to chat with the file path.
