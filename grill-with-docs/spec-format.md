# Spec Format

Specs live in `docs/specs/` as `slug.md` (no numbers — a spec is not a dated record).

A **spec** is the *default* home for a design decision. It states **how one area works today, and the key why**, and it is **edited in place** as the design moves. Reversing a spec is just an edit: no "spec 0002 supersedes spec 0001", no immutable trail, no churn left behind. This is the property that keeps pre-release exploration from accreting into an unreadable pile of superseded records.

Create `docs/specs/` and each spec file lazily — only when there is current truth to write.

## Spec vs. ADR

| | Spec | ADR |
|---|---|---|
| Answers | "How does X work now, and why?" | "Why did we commit to this, knowing the alternatives?" |
| Mutability | Edited in place, freely | Immutable; a new ADR supersedes |
| When it changes | Whenever the design moves | Almost never — that's the point |
| Default? | **Yes** — most decisions live here | No — a rare promotion |

**Default to a spec.** A decision earns an ADR only when you expect to **still hold it at 1.0** *and* reversing it would now be expensive because other things depend on it. Pre-release, almost nothing clears that bar, because almost nothing has stopped moving. When a spec section has genuinely settled and become load-bearing, *then* promote its decision to an ADR and leave the mechanism in the spec. See [adr-format.md](./adr-format.md) for the bar.

The test that catches the common mistake: **if you can imagine reversing this within a few decisions, it is a spec, not an ADR.** Every orbit in a project's history (where state lives, the storage substrate, the data model) was a fluid decision wrongly born into the immutable tier. A spec absorbs that motion invisibly.

## Template

```md
# {Area name}

{One or two sentences naming what this area is and where it lives in the code.}

## {How it works — one section per coherent piece}

{Current truth, stated in the present tense. Fold the *key* rationale inline — the load-bearing "why", not the full debate. A reader orienting before they touch
the code should leave knowing what is true and why it is the way it is.}

## Open questions

{Anything genuinely unsettled, so the next author knows the live edges.
This is where fluid decisions sit before they are resolved into the sections above.}
```

## What goes in a spec

- **Current mechanism.** "The Event log is append-only JSONL under `.workspaces/<id>/`, fronted by an in-memory projection." How it works, now.
- **The load-bearing why, compressed.** "JSONL not SQLite, because status is a projection so there is no hot read path to index." One sentence, not the six-ADR archaeology.
- **The current position of a question that has moved.** Where state lives *now*; what the provenance model *is* — not the history of how it got there (git holds that).
- **Open questions.** The unsettled edges, named.

## What does NOT go in a spec

- **Domain vocabulary** — that is `CONTEXT.md`'s job; a spec uses the terms, it does not define them.
- **A blow-by-blow of rejected alternatives.** Keep the one or two that a reader would otherwise re-propose; the rest is in git and in grill transcripts.
- **A changelog of how the section evolved.** A spec describes the present. Lineage lives in `git log`/`git blame`.
