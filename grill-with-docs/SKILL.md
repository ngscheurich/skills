---
name: grill-with-docs
description: Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates documentation (CONTEXT.md, ADRs) inline as decisions crystallise. Use when user wants to stress-test a plan against their project's language and documented decisions.
---

<what-to-do>

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing.

If a question can be answered by exploring the codebase, explore the codebase instead.

</what-to-do>

<supporting-info>

## Domain awareness

During codebase exploration, also look for existing documentation:

### File structure

Most repos have a single context:

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adrs/                 ← context-specific decisions
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adrs/
```

Create files lazily — only when you have something to write. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adrs/` exists, create it when the first ADR is needed.

## During the session

### Challenge against the ubiquitous language

When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately. "Your ubiquitous language defines 'cancellation' as X, but you seem to mean Y — which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account' — do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"

### Update CONTEXT.md inline — but only for language that already exists

`CONTEXT.md` is **live**: it describes the system as it is _today_. So update it inline only when the term you resolved is about behavior that already exists — a clarification, a sharpening, a renamed concept whose code is already shipped. When a term is resolved, update `CONTEXT.md` right there; don't batch those up. Use the format in [context-format.md](./context-format.md).

When the resolved language describes **unbuilt** work, do **not** edit `CONTEXT.md` yet — doing so makes the live doc assert a model the code does not support, misleading any reader (including an agent you hand the work to). Instead, the agreed wording goes into the brief as the target language, and the `CONTEXT.md` edit becomes an **acceptance criterion** applied when the work lands, in the same change. The language never gets ahead of the code.

`CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, a scratch pad, or a repository for implementation decisions. It captures the ubiquitous language and nothing else.

### Record decisions in a spec by default

When a design decision crystallises, capture it in the relevant **spec** under `docs/specs/` — "how this area works today, and the key why" — edited in place. This is the default home for almost everything. A reversal later is just an edit; it leaves no superseded record behind. Use the format in [spec-format.md](./spec-format.md).

### Promote to an ADR rarely

An ADR is a _promotion_ from the spec tier, not the default. Only offer one when **both**:

1. **The decision has stopped moving** — you expect to **still hold it at 1.0**. If you can imagine reversing it within a few decisions, it is a spec, not an ADR.
2. **It is now expensive to reverse** — other code or decisions depend on it.

If either is missing, it belongs in a spec. The litmus that catches the common mistake: a fluid, still-being-explored decision recorded as an immutable ADR is the exact failure that produces churn and orbits. Use the format in [adr-format.md](./adr-format.md).

</supporting-info>
