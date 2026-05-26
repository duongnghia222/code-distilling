# Distillation Spec: superpowers/subagent-driven-development

**Status:** Draft → awaiting user review.
**Date:** 2026-05-26
**Reference map:** docs/distilling/superpowers-subagent-driven-development-reference-map.md
**Reference path:** /Users/duongnghia/Desktop/nghia/superpowers
**Reference commit:** f2cbfbefebbfef77321e4c9abc9e949826bea9d7
**Source language:** Markdown (skill prose)  →  **Target language:** Markdown (skill prose)

---

> **Source paths in this spec are relative to `Reference path`.** Resolve them with `<REF_PATH>/<source-path>` when reading the file.

## 1. Goal

Replace `code-distilling:distillation-execution` with a lighter-weight, faster execution skill modeled on `superpowers:subagent-driven-development`. Halve the per-chunk dispatch count by collapsing the existing `.t`+`.i` two-task split into a single task that runs `equivalence-tdd` inside the implementer subagent. Simplify the spec-compliance reviewer to SDD's 3-bucket form. Preserve the porting-specific discipline that code-distilling requires: branch safety (no `main`/`master`), mode-aware dispatch (copy / port / learn-then-rewrite), mandatory source-file paste for copy/port, and the final whole-branch reviewer pass. Net effect: noticeably fewer dispatches per plan, smaller prompts per dispatch, fewer re-review iterations, same artifact shape.

## 2. Scope

In scope (replaces the four files of the current `skills/distillation-execution/`):

- `skills/subagent-driven-development/SKILL.md` → `skills/distillation-execution/SKILL.md`
- `skills/subagent-driven-development/implementer-prompt.md` → `skills/distillation-execution/implementer-prompt.md`
- `skills/subagent-driven-development/spec-reviewer-prompt.md` → `skills/distillation-execution/spec-reviewer-prompt.md`
- `skills/subagent-driven-development/code-quality-reviewer-prompt.md` → `skills/distillation-execution/code-quality-reviewer-prompt.md`

The skill name `distillation-execution` and its location are unchanged, so `using-code-distilling` and other code-distilling skills that reference it continue to resolve. No upstream skill changes.

## 3. Mode assignments

| Source chunk | Target file(s) | Mode | Deciding criterion | Adaptation notes |
| ------------ | -------------- | ---- | ------------------ | ---------------- |
| skills/subagent-driven-development/SKILL.md | skills/distillation-execution/SKILL.md | port | Same language but materially different domain content. Section-by-section adaptation required (Integration list, examples, branch-safety positioning, process diagram, model table). Not a copy. | Process diagram: collapse `.t`+`.i` nodes into a single per-task block. Branch safety: keep as top-level gated section (stronger than SDD's red-flag bullet). Integration: rewrite to point at code-distilling sibling skills (`analyzing-reference`, `distillation-design`, `distillation-plan`, `equivalence-tdd`) — drop superpowers refs (`using-git-worktrees`, `writing-plans`, `finishing-a-development-branch`, `executing-plans`, `test-driven-development`, `requesting-code-review`). Example workflow: rewrite in porting vocabulary (copy/port/learn-then-rewrite tasks, not "hook installation script"). Model-selection table: preserve current code-distilling table (mode-aware rows). "Continuous execution" stance: preserve verbatim. Four-status handling (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED): preserve. Final whole-branch reviewer dispatch: preserve. Plan path convention: use `docs/plans/<date>-distill-<repo>-<feature>.md` (not SDD's `docs/superpowers/plans/...`). |
| skills/subagent-driven-development/implementer-prompt.md | skills/distillation-execution/implementer-prompt.md | port | Same language; SDD's shape and tone are the scaffolding, but the prompt must carry the porting-specific payload (equivalence-tdd, mode rules, source paste, hard rules, four-status report block). Materially different content, not a copy. | Use SDD's section ordering and conversational tone. Inject the **Curated Context** block (spec row, reference-map excerpt, source file content for copy/port — OMITTED for learn-then-rewrite, exact test command, fail-state and pass-state expectations, working directory). Replace SDD's soft "Did I follow TDD if required?" with MANDATORY equivalence-tdd: port test → run failing → port impl → run passing → single commit. Keep the hard rules block: do NOT modify the reference path, do NOT read the reference repo, do NOT weaken the test, do NOT skip the failing-state run, do NOT touch files outside the task's named list, do NOT paste reference lines for learn-then-rewrite. Keep the four-status report block (DONE/DONE_WITH_CONCERNS/NEEDS_CONTEXT/BLOCKED) including the DONE template's commit SHA, files list, and failing-then-passing test output lines. Keep mode-specific copy/port/learn-then-rewrite definitions in the equivalence-tdd section. |
| skills/subagent-driven-development/spec-reviewer-prompt.md | skills/distillation-execution/spec-reviewer-prompt.md | copy | SDD's 3-bucket form is being adopted wholesale per design Q3. Adaptations are mechanical (header, status format), not structural. | Replace SDD's "Task tool (general-purpose)" preamble with a direct prompt body (matches code-distilling's existing reviewer-prompt style). Replace SDD's `[FULL TEXT of task requirements]` block with the spec row (Source chunk / Target file(s) / Mode / Deciding criterion / Adaptation notes). Replace SDD's `[From implementer's report]` with `**Implementer commit SHA:** <SHA>` and `**Files touched by the commit:** <LIST>`. Preserve verbatim: the "Do Not Trust the Report" stance, the three buckets (missing requirements / extra-unneeded work / misunderstandings), "Verify by reading code, not by trusting report". Report format: code-distilling conventions `APPROVED` / `ISSUES` (with one-per-line required fixes), so the controller's parser stays unchanged. |
| skills/subagent-driven-development/code-quality-reviewer-prompt.md | skills/distillation-execution/code-quality-reviewer-prompt.md | learn-then-rewrite | The reference is a 25-line pointer at `requesting-code-review/code-reviewer.md`, a superpowers-only file that does not exist in code-distilling. Cannot copy or port — there is no shared template to point at. Must write an inline reviewer prompt that satisfies the behavioral checklist. | Write from understanding of the checklist below; do NOT paste lines from `<REF_PATH>/skills/requesting-code-review/code-reviewer.md`. Behavioral checklist for the rewrite: (a) header takes Task ID, implementer commit SHA, files touched, mode; (b) instructs reviewer to read target project conventions before reviewing (style guide, test framework, sibling files); (c) requires reading code at the SHA, not trusting the implementer's report; (d) reports Strengths / Issues (Critical / Important / Minor) / Assessment; (e) extra porting-specific checks: single-responsibility per file, plan-conformant file layout, file-size growth flagged, mode-appropriate idioms (copy is minimal-diff, port is structure-preserving, learn-then-rewrite reads as native target code); (f) report format `APPROVED` / `ISSUES` matching the spec-reviewer's format. Use the **existing** `skills/distillation-execution/code-quality-reviewer-prompt.md` as a structural sibling — match its section layout so the controller's parsing stays consistent. |

## 4. Target API

The skill's public surface inside the user's project is unchanged in name and shape:

- **Skill name:** `code-distilling:distillation-execution`
- **Skill directory:** `skills/distillation-execution/`
- **Files in directory:** `SKILL.md`, `implementer-prompt.md`, `spec-reviewer-prompt.md`, `code-quality-reviewer-prompt.md`
- **Inputs:**
  - Approved distillation plan at `docs/plans/<date>-distill-<repo>-<feature>.md` (produced by `code-distilling:distillation-plan`).
  - The spec the plan references (`docs/specs/...`).
  - The reference map (`docs/distilling/...`) for paste-excerpts per task.
  - The user's project, on a non-`main`/`master` branch.
- **Process surface:** for each plan task, dispatch one implementer subagent (which runs `equivalence-tdd` internally and commits), then one spec-compliance reviewer, then one code-quality reviewer, looping each reviewer with the implementer until approved. After all tasks: dispatch one whole-branch final reviewer.
- **Outputs:**
  - One git commit per plan task (containing test + implementation, per `equivalence-tdd`).
  - Two reviewer-approved gates per task.
  - One final whole-branch reviewer report.
  - One completion summary to the user (chunks distilled, modes used, tests passing).
- **Required sub-skill:** `code-distilling:equivalence-tdd`, invoked from inside the implementer prompt.
- **Branch safety:** refuses to start on `main`/`master` without explicit user consent.

Upstream consumers (`using-code-distilling` flow chart, `distillation-plan`'s handoff line) continue to point at `distillation-execution` by name. No changes required outside `skills/distillation-execution/`.

## 5. External library plan

| Reference library | Target choice | Notes |
| ----------------- | ------------- | ----- |
| — | — | No external libraries in the reference feature. Both source and target use the harness's `Task`, `TaskCreate`, and `git` only, which are identical across the two skill packages. |

## 6. Hidden coupling resolutions

| Coupling item from ref map | Resolution |
| -------------------------- | ---------- |
| SDD `SKILL.md:131` example plan path hardcoded to `docs/superpowers/plans/feature-plan.md` | Rewrite example to use `docs/plans/<date>-distill-<repo>-<feature>.md` (code-distilling convention; existing `distillation-execution/SKILL.md:190` style). |
| SDD `code-quality-reviewer-prompt.md:11` points at `requesting-code-review/code-reviewer.md` (superpowers-only) | Inline the reviewer prompt (mode = learn-then-rewrite, see §3 row 4). |
| SDD `SKILL.md:268-279` Integration section names superpowers-only sibling skills | Replace with code-distilling skill graph: required upstream = `analyzing-reference` → `distillation-design` → `distillation-plan`; required sub-skill = `equivalence-tdd`; drop worktrees / writing-plans / finishing-a-development-branch / executing-plans / test-driven-development / requesting-code-review references entirely. |
| SDD `SKILL.md:239` branch-safety phrased as a red-flag bullet | Preserve code-distilling's stronger form: top-level **Branch Safety** section with explicit gating ("Never start execution on `main`/`master` without explicit user consent. No exceptions."). |
| SDD `SKILL.md:138-148` example uses domain-foreign vocabulary ("hook installation script", "~/.config/superpowers/hooks/") | Rewrite example in porting domain: a copy chunk and a learn-then-rewrite chunk; show the implementer dispatch, the spec reviewer's APPROVED, the code quality reviewer's ISSUES → re-dispatch → APPROVED, then the final whole-branch reviewer. |
| SDD `implementer-prompt.md:90` "Did I follow TDD if required?" | Harden to "Did I follow `code-distilling:equivalence-tdd` (test first, run failing, port impl, run passing, single commit)?" — no "if required". |

## 7. Equivalence test plan

**Test framework in target:** none — skills are prose. Equivalence is established by a hand-walked behavioral checklist plus a synthetic dry-run, per design Q4.

**Per-chunk test strategy:**

| Target file | Strategy | What the test exercises |
| ----------- | -------- | ----------------------- |
| `skills/distillation-execution/SKILL.md` | spot-check (forced — prose, no automated tests) + dry-run trace | Behavioral checklist items 1–8 below. The synthetic dry-run is the main exerciser of this file. |
| `skills/distillation-execution/implementer-prompt.md` | spot-check + dry-run | Behavioral checklist items 9–14 below. Dry-run surfaces whether the equivalence-tdd block is invoked, source pasting works for copy and is omitted for learn-then-rewrite, and the 4-status report format is preserved. |
| `skills/distillation-execution/spec-reviewer-prompt.md` | spot-check | Behavioral checklist items 15–18 below. Verify 3 buckets are present, "do not trust the report" stance is verbatim, `APPROVED`/`ISSUES` output format matches. |
| `skills/distillation-execution/code-quality-reviewer-prompt.md` | spot-check | Behavioral checklist items 19–23 below. Verify Strengths / Issues (Critical/Important/Minor) / Assessment structure, the extra porting-specific checks, and the report format. |

**Behavioral checklist (the equivalence "test"):**

A human reviewer reads the four ported files and confirms each item. Each item below must be true; an item failing == ISSUES from the spec reviewer in execution.

*SKILL.md:*

1. Per-task block in the process diagram dispatches ONE implementer (not two — `.t` and `.i` are collapsed).
2. Per-task block still shows spec-compliance reviewer running BEFORE code-quality reviewer.
3. **Branch Safety** is a top-level section (not just a red-flag bullet) and refuses to run on `main`/`master`.
4. Model-selection table includes rows for copy / port / cross-language port / learn-then-rewrite / spec-compliance reviewer / code-quality reviewer / final whole-branch reviewer.
5. Four-status handling block (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED) is present and unchanged in semantics.
6. Integration section names only code-distilling skills: `analyzing-reference`, `distillation-design`, `distillation-plan`, `equivalence-tdd`. No `superpowers:` references.
7. Final whole-branch reviewer dispatch is the terminating step after the per-task loop completes.
8. "Continuous execution" stance is preserved: no "Should I continue?" pauses between tasks.

*implementer-prompt.md:*

9. Curated Context block names: spec row (table), reference-map excerpt (deps + hidden coupling), source file content (for copy/port — OMITTED for learn-then-rewrite), exact test command, fail-state expectation, pass-state expectation, working directory.
10. Required Sub-Skill section names `code-distilling:equivalence-tdd` and enumerates its 5 steps (port test, run failing, port impl, run passing, single commit).
11. Hard Rules block forbids: modifying the reference path, reading the reference repo from the subagent, weakening the test, skipping the failing-state run, touching files outside the named list, and pasting reference lines for learn-then-rewrite.
12. Self-Review section asks about equivalence (fail/pass test outputs), mode adherence (copy/port/learn-then-rewrite), and scope.
13. Report Format block specifies DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED with the DONE template including commit SHA, files list, and failing→passing test output lines.
14. Tone matches SDD (conversational, "it is always OK to escalate") — not a wall of rules.

*spec-reviewer-prompt.md:*

15. Three buckets present: missing requirements, extra/unneeded work, misunderstandings.
16. "Do Not Trust the Report" stance present verbatim: read code at the SHA, not the implementer's narrative.
17. Header takes Task ID, implementer commit SHA, the spec row, files touched.
18. Report format: `APPROVED` / `ISSUES` with one-per-line required fixes — parser-compatible with the controller's handling.

*code-quality-reviewer-prompt.md:*

19. Header takes Task ID, implementer commit SHA, files touched, mode.
20. Instructs reviewer to read target project conventions (style guide, test framework, sibling files) before reviewing.
21. Reviewer reads code at the SHA, not the implementer's narrative.
22. Output sections: Strengths / Issues (Critical / Important / Minor) / Assessment.
23. Extra porting-specific checks: single-responsibility per file, plan-conformant layout, file-size growth flagged, mode-appropriate idioms.

**Synthetic dry-run (Plan-fragment fixture):**

Author a 2-chunk fake plan (not committed — sits in the spec or the plan's appendix during validation):

- Chunk A: a `copy` mode task — port a small TypeScript utility (`src/util/clamp.ts`, 30 lines) into the target.
- Chunk B: a `learn-then-rewrite` mode task — independently rewrite a reference's event-bus glue.

Walk the new `distillation-execution` skill against this plan and verify:

- Controller dispatches exactly 2 implementer subagents (one per chunk; no `.t`/`.i` split).
- Chunk A implementer prompt contains the pasted source of `clamp.ts`.
- Chunk B implementer prompt does NOT contain pasted reference source.
- Each chunk goes through: implementer → spec-reviewer (APPROVED) → code-quality-reviewer (APPROVED).
- After both chunks, one whole-branch reviewer dispatch fires.
- No step asks the user "should I continue?" between chunks.

If the trace matches, the port is equivalent.

## 8. Out of scope

- Pulling SDD's `using-git-worktrees` integration. Code-distilling stays worktree-agnostic.
- Pulling SDD's `finishing-a-development-branch` terminal step. Code-distilling's "announce completion with summary" is the terminus.
- Pulling SDD's `executing-plans` parallel-session alternative. Code-distilling assumes same-session execution.
- Pulling SDD's `requesting-code-review/code-reviewer.md` template. The code-quality reviewer prompt is rewritten inline (§3 row 4, mode = learn-then-rewrite).
- Pulling SDD's generic `test-driven-development` skill. Code-distilling already has `equivalence-tdd`, which is the porting-specific analog and is required.
- Any changes to `using-code-distilling`, `analyzing-reference`, `distillation-design`, `distillation-plan`, or `equivalence-tdd`. The replacement is contained to `skills/distillation-execution/`.
- A backward-compatibility shim for callers that depend on the `.t`/`.i` split. Plans produced by `distillation-plan` are the only consumers, and they'll be regenerated.
- Token / time benchmarking. The behavioral checklist + dry-run verifies the skill is structurally lighter; quantitative speedup is not part of the success criteria.
- Source-paste threshold logic (the "hybrid: large files get a path" variant from design Q2). Rejected per Q2 answer.
- Mandatory adaptation-notes check in the spec reviewer (the "hybrid: 3 buckets + adaptation-notes" variant from design Q3). Rejected per Q3 answer.

## 9. Success criteria

- All 23 behavioral checklist items in §7 pass when a reviewer reads the four ported files.
- The synthetic 2-chunk dry-run trace in §7 matches the listed dispatches and artifacts.
- Every hidden-coupling item in §6 has a resolution applied in the corresponding target file.
- `using-code-distilling` and other code-distilling skills continue to resolve `distillation-execution` by name without modification.
- Branch safety still refuses execution on `main`/`master`.
- The final whole-branch reviewer dispatch is preserved as the terminating step.
