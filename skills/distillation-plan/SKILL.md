---
name: distillation-plan
description: Use after an approved distillation spec to produce a task-by-task plan with a source-to-target file map; each implementation task is paired with an equivalence-test task and attribution step
---

# Distillation Plan

Translate the distillation spec into a task-by-task plan ready for execution. The writing-plans analog, specialized for porting.

Assume the engineer (or implementer subagent) knows the target stack but has zero context about the reference repo and questionable taste. Document everything they need: which source path, which target path, which mode, which adaptations, which test to port, what command to run, what passes look like, exact commit message.

**Announce at start:** "I'm using `distillation-plan` to write the implementation plan."

## Inputs

- An approved distillation spec at `docs/specs/YYYY-MM-DD-distill-<repo>-<feature-slug>.md`.
- The reference map the spec was derived from.

If the spec is missing or unapproved, invoke `distillation-design` first.

## Output

A plan at `docs/plans/YYYY-MM-DD-distill-<repo>-<feature-slug>.md` in the user's project, committed to git.

## Checklist

Create one `TaskCreate` entry per item.

1. **Read the spec end-to-end.** Note any chunks where the test strategy or adaptation is unusual — the plan needs explicit guidance there.
2. **Read the reference map.** You will quote paths from it.
3. **Produce the source→target file map** (see structure below).
4. **Order chunks by dependency.** A utility chunk distills before chunks that import it. Identify cycles inside the chunk graph and break them by introducing a target-only seam (record this in adaptation notes).
5. **Write the task list.** Each chunk yields the pair: equivalence-test task → implementation task, both following `equivalence-tdd`. After the chunk, an attribution-header verification step (delegated to `attribution-and-license`'s per-file rules).
6. **Add the final attribution task** at the end of the task list: invoke `attribution-and-license` final pass.
7. **Self-review the plan** (see below).
8. **Commit** with message: `plan: distill <repo>/<feature>`.
9. **Hand off** to `distillation-execution`.

## Plan document header

Every plan starts with:

```markdown
# Distillation Plan: <repo>/<feature>

> **For agentic workers:** REQUIRED SUB-SKILLS: Use `code-distilling:distillation-execution` to execute this plan task-by-task, which dispatches subagents that follow `code-distilling:equivalence-tdd` for every implementation task. Steps use checkbox (`- [ ]`) syntax.

**Spec:** `docs/specs/YYYY-MM-DD-distill-<repo>-<feature-slug>.md`
**Reference map:** `docs/distilling/<repo>-<feature-slug>-reference-map.md`
**Reference commit:** <SHA>
**Source language → Target language:** <lang> → <lang>

---
```

## Source→target file map

A table covering every distilled file. One row per target file.

```markdown
## Source → Target File Map

| # | Source path(s) | Target path | Mode | Test source | Adaptation notes |
|---|----------------|-------------|------|-------------|------------------|
| 1 | ref-code/<repo>/src/util/lru.ts | src/cache/lru.ts | copy | ref-code/<repo>/test/util/lru.test.ts → test/cache/lru.test.ts | rename `LRU` to `LruCache` per target conventions |
| 2 | ref-code/<repo>/src/auth/oauth.ts | src/auth/oauth.ts | port | ref-code/<repo>/test/auth/oauth.test.ts → test/auth/oauth.test.ts | callback-style → async/await; swap `axios` for `fetch` |
| 3 | (design influence) ref-code/<repo>/src/auth/strategy.ts | src/auth/strategy.ts | learn-then-rewrite | fresh-equivalence-tests captured in spec §7 | independent implementation |
| ... | ... | ... | ... | ... | ... |
```

For learn-then-rewrite rows, the source column names the reference module that inspired the design, prefixed with `(design influence)`.

## Task structure

Tasks come in pairs (test → impl) per chunk, followed by a brief attribution verification step. Each task is 2–5 minutes of focused work.

### Task N.t: Port test for <chunk>

**Files:**
- Create: `test/path/to/chunk.test.<ext>`
- Read: `ref-code/<repo>/test/path/to/chunk.test.<ext>` (or "behavior captures in spec §X" for fresh-equivalence-test)

**Mode:** copy / port / learn-then-rewrite (test-side translation if different)

- [ ] **Step 1: Port/translate the test file**

```<lang>
<paste the actual ported/translated test code here>
```

- [ ] **Step 2: Run the test against the not-yet-ported target**

Run: `<target test command, e.g. pnpm test test/path/chunk.test.ts>`
Expected: FAIL with `Cannot find module 'src/path/chunk'` or similar.

- [ ] **Step 3: Confirm failure is for the right reason**

If the test passes accidentally or fails for an unrelated reason, stop and fix the test before moving to the implementation task.

### Task N.i: Port implementation for <chunk>

**Files:**
- Create or modify: `src/path/to/chunk.<ext>`
- Read: `ref-code/<repo>/src/path/to/chunk.<ext>`
- Test: `test/path/to/chunk.test.<ext>`

**Mode:** copy / port / learn-then-rewrite

**Adaptation notes from spec:** <quote the relevant row>

- [ ] **Step 1: Add the attribution header**

```<comment style>
<paste the actual header here with values filled in>
```

- [ ] **Step 2: Port/translate/rewrite the implementation**

```<lang>
<paste the implementation here — full code, no placeholders>
```

(For `learn-then-rewrite`: do NOT paste the reference code. Instead, paste a guidance note: "Write a target-idiomatic implementation that satisfies the test in N.t. Reference is design influence only; do not consult its source lines while implementing.")

- [ ] **Step 3: Run the test**

Run: `<target test command>`
Expected: PASS.

- [ ] **Step 4: Commit**

```bash
git add test/path/to/chunk.test.<ext> src/path/to/chunk.<ext>
git commit -m "distill(<repo>): <chunk summary>

Source: <repo>@<short-sha>:<source path>
License: <SPDX>"
```

(For learn-then-rewrite, use `Source-influence:` instead of `Source:`.)

## Final task

```markdown
### Task F: Finalize attribution

**Sub-skill:** `attribution-and-license` — final pass.

- [ ] Verify every distilled file has a header (grep across the distillation branch).
- [ ] Generate or update `ATTRIBUTION.md` with a section for `<repo>` listing every distilled file.
- [ ] Copy the source LICENSE to `licenses/<repo>-<spdx>.txt` if not already present.
- [ ] Verify every distillation commit has a `Source:` or `Source-influence:` trailer.
- [ ] Commit: `chore(attribution): finalize attribution for <repo> distillation`.
```

## No placeholders

These are plan failures — never write them:

- "TBD", "TODO", "implement later", "fill in details".
- "Add appropriate error handling" / "handle edge cases" without showing how.
- "Port the test" without including the actual ported test code.
- "Similar to Task N" — repeat the code, the implementer may read tasks out of order.
- Steps that describe what to do without showing how (code blocks required where code is involved).
- Source paths that don't exist in `ref-code/<repo>/`.
- Target paths that don't follow the user's project conventions (check the project's existing layout).

## Self-review

After writing the plan, with fresh eyes:

1. **Spec coverage:** every chunk from the spec's mode-assignments table is in the file map and has a task pair. Anything from Section 9 ("Out of scope") of the spec is NOT in the plan.
2. **Path consistency:** every source path exists in `ref-code/<repo>/`. Every target path is one a reasonable implementer would create.
3. **Mode consistency:** the mode column in the file map matches what the task prompts implementer to do (copy code shown vs. design-influence note).
4. **Test source consistency:** if the spec says `port-reference-test` for a chunk, the test task references a real test file in `ref-code/`. If `fresh-equivalence-test`, the test task pastes the cases from the spec.
5. **Attribution coverage:** every implementation task has a step-1 attribution header. The final task is present.
6. **Dependency order:** for each chunk, every chunk it imports is earlier in the task list.

Fix issues inline. If a chunk is missing a task pair, add it. If the spec has been over- or under-implemented, escalate to the user before continuing.

## Commit and hand off

After self-review, commit the plan:

```bash
git add docs/plans/YYYY-MM-DD-distill-<repo>-<feature-slug>.md
git commit -m "plan: distill <repo>/<feature>"
```

Announce:

> "Plan written and committed to `<path>`. Ready for execution. Invoking `distillation-execution`."

## What you do NOT do

- You do **not** write the actual ported code into the target tree. That's execution.
- You do **not** re-decide modes; modes come from the spec.
- You do **not** modify `ref-code/`.
