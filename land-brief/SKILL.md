---
name: land-brief
description: Clean up locally after an agent's brief branch merges on GitHub — update main, mark the brief done, and tear down branches (and the worktree, when it is a legacy .worktrees/ one). Use when a remote branch worked in a worktree (a tool-managed workspace worktree outside .worktrees/, or legacy .worktrees/<slug>) has merged and the user says "land <slug>", "land the merged brief", "clean up after the merge", or otherwise wants post-merge cleanup.
---

# Land a brief

The counterpart to `/begin-brief`: once an agent's work has **merged on GitHub**, settle the local state it leaves behind. See `CONTEXT.md` for the ubiquitous language and `AGENTS.md` for conventions.

This skill follows the **GitHub-PR landing path** that the project actually uses today — a hand-edited done-marking commit (release tagging and changelog updates are currently **suspended** — see Step 4). ADR-0008 describes a future automated ship flow that would own the brief `status:` write automatically; it isn't built yet, so we do it by hand.

## Operate from the primary checkout

Every step runs against `main` in the **primary checkout** at the project root (`git worktree list` — the entry checked out at the project root itself, not nested under `.worktrees/` or the managed workspace directory). Two reasons: `main` can only be checked out in one worktree, and **you cannot remove a worktree you are standing in**. So `cd` into the primary checkout **once** at the start, then run plain `git …` from there — every command below assumes that cwd.

## Step 1 — Confirm the merge and gather facts

Don't clean up work that hasn't actually merged. Establish:

- **The brief / slug** — the branch is named `<slug>`, the brief is `briefs/<slug>.md`. The worktree path **varies** — find it in `git worktree list`. A tool-managed workspace keeps its worktree in the managed workspace directory, outside `.worktrees/`; the legacy hand-made layout is `.worktrees/<slug>`. This distinction decides the teardown in Step 5.
- **The merge happened** — `gh pr view <slug> --json state,mergedAt,mergeCommit,number`. Confirm `state: MERGED`. Note the **PR number** and **merge commit** SHA.

If it hasn't merged, stop and say so.

## Step 2 — Update main

```
git switch main
git pull --ff-only
```

The merge commit should now be in `main`'s history. Verify: `git log --oneline -1 <mergeCommit>`.

## Step 3 — Mark the brief done

Per ADR-0008 the canonical status is `done` once shipped. Hand-edit `briefs/<slug>.md` frontmatter `status:` → `done` (it is usually `ready` or `in_progress`), leaving the rest of the file untouched. Then commit with an explicit pathspec (see house style; the index may hold unrelated WIP):

```
git commit briefs/<slug>.md -m "chore(briefs): mark <slug> done"
```

Use the `/commit` style for the message. If landing the brief unblocks dependents, you may add their files to the same commit and widen the subject (e.g. `docs(briefs): mark <slug> done, add <follow-up>`).

## Step 4 — Release tracking (suspended)

> **Suspended until the `0.1.0` candidate.** Tagging and changelog updates are paused: releasing is now at the developer's discretion, with no per-brief cadence (see ADR-0006). Do **not** edit `CHANGELOG.md` or create a tag — skip to Step 6. The original procedure is preserved, commented out below, for when tracking resumes (uncomment Steps 4–5 and restore the tag lines in Steps 6–7).

<!-- SUSPENDED — release tracking resumes at the 0.1.0 candidate.

## Step 4 — Update the changelog

`CHANGELOG.md` follows [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/). Cut the work being landed from `[Unreleased]` into a dated version section. **Decide the version first**, since both this step and the tag below depend on it:

1. Find the latest tag: `git tag --sort=-creatordate | head -1`.
2. **Sweep for untagged features** — `git log <last-tag>..main --oneline`. More than one feature can slip through; each **shippable** one gets its own version (minor bump, oldest first) and its own changelog section. Non-shippable work (see the shippable-product rule below) is **not** a release: it consumes no version number, so don't leave a hole for it — the next shippable feature takes the next version.
3. **Reconcile tags against the log** — the inverse drift: a release that was tagged but never recorded (a direct fix+tag straight to `main` bypasses this skill, so its entry is missing and sits *behind* the latest tag where the sweep above can't see it). Diff the two lists: `git tag --sort=v:refname` against the `## [X.Y.Z]` sections already in `CHANGELOG.md`. Backfill a section (and its compare link) for every tag with no entry, dated from the tag (`git for-each-ref --format='%(creatordate:short)' refs/tags/<tag>`) and worded from its commit. Skip tags deliberately left out (none should be, but say so if you do).
4. For each version `vX.Y.0`, add a `## [X.Y.0] - <date>` section **above the previous version**, where `<date>` is the merge/release date (`YYYY-MM-DD`, from `gh pr view`'s `mergedAt` or today). Move any matching items out of `[Unreleased]` and write them as user-facing prose grouped under the relevant Keep a Changelog headings — `### Added`, `### Changed`, `### Deprecated`, `### Removed`, `### Fixed`, `### Security` (omit empty ones). Translate the PR's user-facing effect, not the commit subjects; keep house-style domain terms; link any ADR the brief cites.
5. At the bottom, add the compare link `[X.Y.0]: https://github.com/<org>/<repo>/compare/<prev-tag>...vX.Y.0` and repoint `[Unreleased]` to `vX.Y.0...HEAD` (use the **newest** tag created).
6. Fold `CHANGELOG.md` into the done-marking commit from Step 3 if it is not yet pushed (`git commit --amend`); otherwise commit it on its own:
   ```
   git commit CHANGELOG.md -m "docs(changelog): record vX.Y.0"
   ```

**The changelog and the tag go hand in hand: a release earns both or neither.** Both record **shippable product** changes only — what a user of the CLI and server gets. If a landed brief has no user-facing product effect, say so and **skip both the changelog entry and the tag** (and the version number — see Step 4.2); do not invent an entry, and do not tag "just to mark it". This covers pure internal refactors, CI, and build changes, and also **non-shippable tooling that lives in the repo but ships nothing to users** — agent skills (`.agents/skills/`), `.claude/` config, repo automation. A new skill is notable to contributors but it is not a product change; leave it untagged and unlogged. (Docs are a judgment call: a docs brief that reshapes user-facing documentation can still earn a release.) The brief is still marked `done` (Step 3) and its branches torn down (Step 6) — only the version artifacts are skipped.

## Step 5 — Tag the release (and sweep for stragglers)

Releases are **one annotated tag per shipped feature**, minor bump, message = a single imperative one-line feature description reading like a brief title. Use the version(s) you settled on in Step 4 — and **only** the shippable ones: a brief you skipped in the changelog (non-shippable work, per the rule above) gets **no tag** either.

1. Tag the **PR merge commit** (not the done-marking or changelog chore commit):
   ```
   git tag -a vX.Y.0 <mergeCommit> -m "<one-line feature description>"
   ```
   Inspect prior style with `git tag -n1 <last-tag>`. Verify it is annotated: `git cat-file -t vX.Y.0` → `tag`.

-->

## Step 6 — Push, then tear down

```
git push origin main
# Release tagging is suspended (see Step 4); no tags to push until the 0.1.0 candidate.
```

Then tear down (raw git — a managed workspace-remove command isn't built). **The worktree teardown is conditional on where the worktree lives** (the path you found in Step 1):

- **Legacy `.worktrees/<slug>` worktree** — remove it and the local branch:
  ```
  git worktree remove .worktrees/<slug>      # add --force if it complains of changes you've confirmed are safe to drop
  git branch -d <slug>                        # -d refuses if unmerged; if the PR squash-merged, confirm then use -D
  ```
- **Tool-managed worktree (its worktree lives in the managed workspace directory, outside `.worktrees/`)** — **skip the worktree removal and the local `git branch -d`.** That workspace is owned by the tool (or the user); leave it in place for them to remove. The branch is still checked out there, so deleting it would fail anyway.

The remote-side cleanup runs the same way regardless of worktree layout:

```
git push origin --delete <slug>             # skip if GitHub already auto-deleted the head branch
git fetch --prune                            # drop the stale remote-tracking ref
```

## Step 7 — Verify

Confirm the end state and report it plainly:

- `git worktree list` — the `<slug>` worktree is gone (only if it was a legacy `.worktrees/` worktree you removed; a tool-managed workspace is expected to remain).
- `git branch -a | grep <slug>` — no remote-tracking branch remains; the local branch is also gone unless you intentionally left a tool-managed worktree in place.
- `briefs/<slug>.md` reads `status: done`.
- Release tracking is suspended (Step 4): no tag was created and `CHANGELOG.md` was left untouched.

## Notes

- **Squash vs merge commit.** The convention tags the merge commit. If a PR was squash-merged, `git branch -d` will refuse (the branch isn't an ancestor) — confirm the work landed, then `-D`, and point the tag at the squash commit on `main`.
- **Don't invent a version.** If the latest tag is ambiguous or `<last-tag>..main` shows features you don't recognize, surface them and ask before tagging.
