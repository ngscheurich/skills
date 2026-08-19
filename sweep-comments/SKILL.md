---
name: sweep-comments
description: A procedure for evaluating and trimming code comments across the codebase. Use the user says "sweep comments", "check for comment quality", or otherwise wants an agent to clean up code comments.
---

# Run a comment sweep

> [!IMPORTANT]
> The _rules_ live in the project's prose style guide; this is the _process_ that applies them at scale without drifting off the bar.

A sweep cuts comments that don't earn their place. The hard part is not knowing the rules — it's holding a consistent, aggressive altitude across dozens of files. This guide exists because the rules alone can underdetermine that altitude and settle too timid.

---

## 1. The core principle

A comment exists to protect the next editor from breaking something **they cannot see in the code**. That is the whole job. Apply one litmus to every comment:

> **Would the next editor break something they can't see from the code? If no, the comment should not exist.**

Apply it strictly. "It helps you skim," "it's a nice summary," "it's conceptually relevant" — none of these pass. Cut harder than feels safe; the instinct to keep is almost always too generous.

---

## 2. The per-comment decision procedure

Walk every comment through this. Most land on "delete."

1. **Does it document an external tool's behavior?** (What the language runtime, the OS, git, a library, or a separate service does.) → Rewrite to state _our_ choice and its consequence, or delete. _"Without `IS_SANDBOX=1` the hands-off launch exits at startup"_ — not a paragraph on why the external tool refuses to run as root.

2. **Does the function body show it within a few lines?** → Delete. The flags a branch passes, the default a value resolves to, "returns `None` on failure" right above `Error(_) -> None` — all visible, all cut.

3. **Is the item private / unexported?** → Default to **delete**. Keep a one-line comment _only_ for a genuine gotcha the code cannot show: an ordering dependency, a deliberate absence, a footgun an editor would trip over. **Never describe what a private function does** — the name carries that. If you're restating the name, delete.

4. **Is the comment there to translate a name?** (A unit, what it returns, what the constructor means.) → The name is wrong. Note it for a rename pass; delete the comment. Don't rename mid-sweep — see §6.

5. **Does it cite a doc/ADR, recount history, or describe the roadmap?** → Delete those parts. No `docs/adrs/…`, no bare `ADR-0007`, no "this once dropped…", no "retired X," no "will become."

### A worked anchor

Before, on a private function:

```
// The checks `spirit doctor` examines, in report order: doctor's baseline
// unioned with each active Provider's declared checks, plus the config
// Safeguard for `[git] run_hooks`... v1 ships one of each Provider... Claude
// Code's `claude` is deliberately absent: the Agent runs inside the Sandbox,
// so it lives in the OCI image, not on the Host PATH. The Agent Provider's
// Host check is instead the server's OAuth token source...   (12 lines)
```

After:

```
// `claude` is deliberately absent: the Agent runs inside the Sandbox (in the
// OCI image), not on the Host PATH. The Provider's Host check is the server's
// OAuth token source instead.
```

Three lines survive, and only because they stop an editor from "fixing" a perceived omission by adding a `claude` PATH check. "In report order" and the union are visible in the body; "v1 ships one Provider" is rationale, not a footgun; the env-var name is on the call path already.

---

## 3. Hard constraints

- **Only touch comments.** Code tokens stay byte-identical. (Formatters may re-align whitespace as a _consequence_ of removing a trailing comment — that's fine; tokens are unchanged.)
- **Never modify runtime string literals** — user-facing messages, help text, embedded shell scripts, protocol text — even when they contain an ADR reference. They are data, not prose.
- **Don't rename in the sweep.** A rename is a code change with its own blast radius. Record bad names; do them as a separate follow-up pass (§6).

---

## 4. The workflow

### Scope

Count comment lines per file and sort by volume — the worst offenders are where the bar matters most and where calibration pays off.

```sh
for f in $(find <src> -name '*.<ext>'); do
  echo "$(grep -cE '^\s*//' "$f") $f"
done | sort -rn
```

### Calibrate by hand first

Sweep the single worst offender yourself, before delegating anything. Get a human to sign off on that file. This is not optional — it converts the abstract bar into a concrete altitude everything else measures against. Expect to re-cut it two or three times as the human pushes you lower; the corrections _are_ the calibration. Commit the approved file as a reference.

### Batch and delegate

Group the remaining files into batches (by subsystem reads well). For each batch, hand each file-group to a subagent with: the rules (the §-references above), the _committed reference files_ (one aggressive, one moderate — they show the target altitude better than prose), and the hard constraints. Tell each subagent to **edit and format only — never build or test**, because concurrent builds race. Have it report per-file before/after counts and flag any borderline call.

### Gate every batch

Before committing a batch, run all three gates from the parent (not the subagents):

1. **Build** — the project compiles.
2. **Test** — the full suite passes.
3. **Comment-only proof** — strip comments and whitespace from both sides and diff; it must be empty. Use `bash -c` explicitly (don't rely on the interactive shell's word-splitting):

```sh
bash -c '
norm() { sed -E "s://.*$::" | tr -s "[:space:]" " " | grep -vE "^ *$"; }
for f in $(git diff --name-only -- "<glob>"); do
  d=$(diff <(git show HEAD:"$f" | norm) <(norm < "$f"))
  [ -n "$d" ] && { echo "REAL CODE DIFF in $f"; echo "$d"; }
done
echo done'
```

If a gate fails, fix before committing. Commit per batch with a message that records what was cut and that code is verified unchanged.

### Checkpoint with the human

Decide up front, with the human, how much review they want per batch — full-diff or spot-check — and whether to keep delegating. Surface genuine policy forks (e.g. "do self-evident variant docs keep a one-liner?") as questions rather than guessing; the answer applies to every remaining file, so resolving it once prevents rework. When a ruling lands, **fold it back into the style guide and your memory** so the next sweep starts already calibrated.

---

## 5. Pitfalls

- **Don't chase the comment-line count.** The count includes `///`/trailing-blank lines and per-member docs, so a file that cut a 20-line essay to 4 lines may show a small delta. Judge by what's left, not by the number. A 17% drop on a file that was mostly legitimate public docs is the _right_ result, not timidity — the Claude Code adapter fell 91% because it was uniquely bad (documenting an external tool at length); domain files with real invariants fall far less.
- **Formatter realignment is benign.** Removing a trailing field comment lets the formatter re-align the struct block. The verification in §4 ignores whitespace, so it won't flag this — but a naive line diff will. Verify tokens, not columns.
- **Shell word-splitting bites.** A `files="a b c"; for f in $files` loop silently runs once on the whole string in `fish` and other non-POSIX shells, producing a false "all clear." Always run verification loops through `bash -c`.
- **Per-language doc conventions differ.** Gleam `///` wants a trailing blank line and per-variant docs; Go wants the identifier-name-first form and no trailing blank. Don't carry one language's rule into the other.
- **The merge after a long sweep.** If the base branch moved, a conflict in a swept file is usually "their new code + your comment treatment." Take their functionality, apply your trimming to it — don't re-sweep their new code wholesale inside a conflict resolution.

---

## 6. Follow-up: names the comments were propping up

During the sweep you'll find comments that exist only because a name is weak (`callError` that actually _shapes_ an error; a passthrough wrapper that adds nothing). Don't fix these mid-sweep. Afterward, run a small, separate refactor pass: rename so the name carries what the comment did, then delete the now-redundant comment. Keep it scoped, build-and-test gated, and committed apart from the sweep — it's a code change, not a comment change.
