---
name: to-prd
description: Turn the current conversation context into a PRD and write it to docs/prds/<name>.md. Use when the user wants to capture a feature or initiative as a PRD from the current context.
---

# To PRD

Take the current conversation context and codebase understanding and produce a PRD as a markdown file under `docs/prds/`. Do NOT interview the user — synthesize what you already know.

## Process

1. Explore the project to understand the current state of the code, if you haven't already. Use the project's ubiquitous language from `CONTEXT.md` throughout the PRD, and respect any ADRs in the area you're touching.

2. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

   A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

   Check with the user that these modules match their expectations. Check with the user which modules they want tests written for.

3. Write the PRD using the template below to `docs/prds/<name>.md`, where `<name>` is a short kebab-case slug describing the feature (e.g. `docs/prds/onboarding.md`). If the user has a preferred slug, use that. Don't shadow an existing file — if `<name>.md` already exists, ask for a different slug.

4. Suggest `/to-briefs docs/prds/<name>.md` as the next step to break the PRD into individually-grabbable **briefs**.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a developer using the system, I want to see all my briefs in a kanban view, so that I can see what's in flight at a glance </user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the user
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the project)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
