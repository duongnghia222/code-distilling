# Dry-Run Trace: superpowers/subagent-driven-development → distillation-execution

Equivalence check for the port. Walks the newly-ported `distillation-execution`
skill against a synthetic 2-chunk distillation plan and records what the
controller would dispatch. Pass if the trace matches every bullet in spec §7's
"Synthetic dry-run" section.

**Skill under test:** `skills/distillation-execution/` (commits 4780aef → 7deac9a on branch `distill/superpowers-sdd`).
**Spec reference:** `docs/specs/2026-05-26-distill-superpowers-subagent-driven-development.md` §7.

---

## Synthetic plan fixture

A pretend distillation plan with two chunks. Not committed anywhere outside this trace.

```markdown
# Distillation Plan: awesome-libs/utility-and-bus

**Spec:** docs/specs/2026-05-26-distill-awesome-libs-util-bus.md
**Reference path:** /Users/me/code/awesome-libs
**Reference commit:** deadbeefcafe1234
**Source language → Target language:** TypeScript → TypeScript

## Source → Target File Map

| # | Source path | Target path | Mode | Test source | Adaptation notes |
|---|-------------|-------------|------|-------------|------------------|
| 1 | src/util/clamp.ts | src/util/clamp.ts | copy | test/util/clamp.test.ts → test/util/clamp.test.ts | rename namespace `awesome` → project default |
| 2 | (design influence) src/bus/event-bus.ts | src/bus/event-bus.ts | learn-then-rewrite | fresh-equivalence-tests captured in spec §7 | bus stays in-process; drop DI-container coupling |

## Task 1: Port src/util/clamp.ts
... (clamp body, test, etc.) ...

## Task 2: Rewrite src/bus/event-bus.ts
... (design-influence note, fresh tests captured, no source paste) ...
```

The synthetic plan deliberately exercises both code paths the port cares about: a `copy` chunk with a pasteable source file, and a `learn-then-rewrite` chunk where the source must not leak into the implementer's prompt.

---

## Controller trace

The controller is the harness running `code-distilling:distillation-execution` against the fixture. The trace below is what `skills/distillation-execution/SKILL.md` (as ported on this branch) instructs the controller to do.

### Setup

```
Controller: I'm using distillation-execution to execute the plan with subagent-driven dispatch.

[Read plan once: the synthetic plan above]
[Note Reference path = /Users/me/code/awesome-libs]
[Extract Task 1 (copy, clamp.ts) and Task 2 (learn-then-rewrite, event-bus.ts)]
[Create TaskCreate entries: Task 1, Task 2]
```

Per SKILL.md `## Steps` items 1–2. One task per chunk — no `.t`/`.i` split.

### Task 1: copy chunk (clamp.ts)

Controller builds the implementer prompt by filling in `implementer-prompt.md`'s placeholders:

```
[Dispatch implementer subagent with:]

  Task: <verbatim text of plan's Task 1, including step list>

  Curated Context:
    Spec row: | src/util/clamp.ts | src/util/clamp.ts | copy | same lang, simple util | rename namespace |
    Reference map excerpt: (clamp's deps + hidden coupling: none)
    Source file content:
      ```ts
      export function clamp(n: number, lo: number, hi: number): number {
        return Math.min(Math.max(n, lo), hi);
      }
      ```                      ← SOURCE PASTED (mode = copy)
    Test command: pnpm test test/util/clamp.test.ts
    Fail-state: "Cannot find module 'src/util/clamp'"
    Pass-state: test exits 0
    Working directory: /Users/me/projects/target

[Implementer runs equivalence-tdd internally:]
  - Ports test/util/clamp.test.ts. Runs it. FAIL: Cannot find module 'src/util/clamp'.
  - Ports src/util/clamp.ts (namespace renamed). Runs test. PASS.
  - Commits test + impl in one commit (e.g., a1b2c3d).
  - Returns DONE with SHA + files + failing→passing test output lines.

[Controller dispatches spec compliance reviewer]
  Spec reviewer fills spec-reviewer-prompt.md with:
    Task ID, implementer SHA a1b2c3d, files touched, spec row.
  Spec reviewer reads the diff at a1b2c3d.
  Spec reviewer reports APPROVED — three buckets clean.

[Controller dispatches code quality reviewer]
  Code quality reviewer fills code-quality-reviewer-prompt.md with:
    Task ID, implementer SHA a1b2c3d, files touched, mode = copy,
    target project conventions (TS strict, vitest, sibling utils in src/util/).
  Code reviewer reads the diff at a1b2c3d.
  Code reviewer reports APPROVED.

[Mark Task 1 complete]
```

Per SKILL.md `## Process` per-task block and `## Curated Subagent Context`. Source pasted because mode = copy.

### Task 2: learn-then-rewrite chunk (event-bus.ts)

Controller proceeds **immediately** to Task 2 — no "should I continue?" pause. Per SKILL.md "Continuous execution" stance.

```
[Dispatch implementer subagent with:]

  Task: <verbatim text of plan's Task 2>

  Curated Context:
    Spec row: | (design influence) src/bus/event-bus.ts | src/bus/event-bus.ts | learn-then-rewrite | DI-container coupling absent in target | drop DI; in-process pub/sub only |
    Reference map excerpt: (bus's deps + hidden coupling: DI-container, lifecycle hooks)
    Source file content: <OMITTED — mode = learn-then-rewrite>      ← NO SOURCE PASTE
    Test command: pnpm test test/bus/event-bus.test.ts
    Fail-state: "Cannot find module 'src/bus/event-bus'"
    Pass-state: test exits 0
    Working directory: /Users/me/projects/target

[Implementer runs equivalence-tdd internally:]
  - Writes test/bus/event-bus.test.ts from spec §7's captured cases.
  - Runs it. FAIL on missing module.
  - Writes an INDEPENDENT src/bus/event-bus.ts. Does NOT consult the reference's source lines.
  - Runs test. PASS.
  - Commits in one commit (e.g., e4f5g6h).
  - Returns DONE.

[Controller dispatches spec compliance reviewer]
  Spec reviewer reads diff at e4f5g6h, checks the three buckets.
  Specifically: under "Misunderstandings", reviewer looks for reference-source phrasing.
  Reports APPROVED.

[Controller dispatches code quality reviewer]
  Code reviewer fills the template with mode = learn-then-rewrite.
  Under check 11 ("Mode-appropriate idioms"), reviewer specifically scans
  for suspiciously source-like phrasing.
  Reports APPROVED.

[Mark Task 2 complete]
```

Per SKILL.md `## Curated Subagent Context`: "omit for learn-then-rewrite to enforce independence". Per Red Flag: "Let the implementer pull lines from the reference for a learn-then-rewrite chunk — Never."

### Termination

```
[No more tasks]
[Dispatch final code reviewer for the whole branch diff against the base]
  Reviewer reads the cumulative diff (a1b2c3d + e4f5g6h vs. base).
  Reports APPROVED.

[Controller announces completion:]
  "Done. Chunks distilled: 2 (1 copy, 1 learn-then-rewrite). Tests passing.
   No concerns."
```

Per SKILL.md `## Steps` item 4 and `## Completion`.

---

## Equivalence check against spec §7 dry-run bullets

| Spec §7 bullet | Trace satisfies? | Evidence in trace |
|----------------|------------------|-------------------|
| Controller dispatches exactly 2 implementer subagents (one per chunk; no `.t`/`.i` split). | ✅ | Task 1 dispatches one implementer (a1b2c3d). Task 2 dispatches one implementer (e4f5g6h). Total: 2. |
| Chunk A implementer prompt contains the pasted source of `clamp.ts`. | ✅ | "Source file content: ... ← SOURCE PASTED (mode = copy)" in Task 1 block. |
| Chunk B implementer prompt does NOT contain pasted reference source. | ✅ | "Source file content: \<OMITTED — mode = learn-then-rewrite\> ← NO SOURCE PASTE" in Task 2 block. |
| Each chunk goes through: implementer → spec-reviewer (APPROVED) → code-quality-reviewer (APPROVED). | ✅ | Both Task 1 and Task 2 traces show the implementer → spec-reviewer (APPROVED) → code-quality reviewer (APPROVED) sequence. |
| After both chunks, one whole-branch reviewer dispatch fires. | ✅ | "Dispatch final code reviewer for the whole branch diff against the base" in the Termination block. |
| No step asks the user "should I continue?" between chunks. | ✅ | Trace moves directly from "Mark Task 1 complete" into "Task 2: learn-then-rewrite chunk", with the in-text note "no 'should I continue?' pause". |

All six bullets pass. The port is equivalent to the spec's intent.

---

## Notes for the human reviewer

- The implementer and reviewer dispatches in this trace are illustrative — actual SHAs would be produced by real git commits during execution. What matters for equivalence is the **shape** of each dispatch (which prompt template, which placeholders filled, which content pasted vs. omitted), and the trace shows that shape exactly as the ported `SKILL.md` instructs.
- If a future revision of the skill changes any of these six behaviors, this trace will need to be re-walked.
