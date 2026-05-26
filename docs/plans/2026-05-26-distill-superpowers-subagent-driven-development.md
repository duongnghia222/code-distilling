# Distillation Plan: superpowers/subagent-driven-development

> **For agentic workers:** REQUIRED SUB-SKILLS: Use `code-distilling:distillation-execution` to execute this plan task-by-task. The execution skill dispatches subagents that follow `code-distilling:equivalence-tdd` for every implementation task. Steps use checkbox (`- [ ]`) syntax.

**Spec:** `docs/specs/2026-05-26-distill-superpowers-subagent-driven-development.md`
**Reference map:** `docs/distilling/superpowers-subagent-driven-development-reference-map.md`
**Reference path:** /Users/duongnghia/Desktop/nghia/superpowers
**Reference commit:** f2cbfbefebbfef77321e4c9abc9e949826bea9d7
**Source language → Target language:** Markdown → Markdown

> **Path convention:** source paths in this plan are relative to `Reference path`. The execution skill resolves them with `<REF_PATH>/<source-path>` when reading files.

---

## Deviations from the standard plan template

This port is markdown prose, not executable code. Two documented deviations:

1. **One task per chunk, not `.t`+`.i` pairs.** The feature has no automated test files to port. The spec's §7 makes equivalence a hand-walked behavioral checklist plus a synthetic dry-run. Pairing tasks against a non-existent test buys nothing. This also matches the spec's design Q1 answer (the very behavior we're porting in).
2. **File contents are not pasted inline.** For prose ports, duplicating the full target file in the plan adds no audit value — the file map below names every target, and the spec's §3 row plus §7 checklist subset is the implementer's spec. The implementer writes the file content directly into the target tree.

The plan still names every target file, its mode, the spec row anchor, and the per-chunk §7 checklist items that serve as the equivalence test.

---

## Source → Target File Map

Source paths are relative to `Reference path` (see plan header). All four files replace existing files in `skills/distillation-execution/`.

| # | Source path | Target path | Mode | Test source | Adaptation notes |
|---|-------------|-------------|------|-------------|------------------|
| 1 | skills/subagent-driven-development/SKILL.md | skills/distillation-execution/SKILL.md | port | spec §7 checklist items 1–8 (SKILL.md row) | spec §3 row 1 |
| 2 | skills/subagent-driven-development/implementer-prompt.md | skills/distillation-execution/implementer-prompt.md | port | spec §7 checklist items 9–14 (implementer row) | spec §3 row 2 |
| 3 | skills/subagent-driven-development/spec-reviewer-prompt.md | skills/distillation-execution/spec-reviewer-prompt.md | copy | spec §7 checklist items 15–18 (spec-reviewer row) | spec §3 row 3 |
| 4 | (design influence) skills/subagent-driven-development/code-quality-reviewer-prompt.md | skills/distillation-execution/code-quality-reviewer-prompt.md | learn-then-rewrite | spec §7 checklist items 19–23 (code-quality row) | spec §3 row 4 |

Dependency order: tasks 1–4 are independent (each rewrites one file in the same directory). Recommended order is the file map order: SKILL.md first (defines the shape the prompt files plug into), then the three prompt files, then the synthetic dry-run task last.

---

## Task 1: Port `SKILL.md`

**Files:**
- Modify: `skills/distillation-execution/SKILL.md`
- Read: `<REF_PATH>/skills/subagent-driven-development/SKILL.md`
- Read for context: existing `skills/distillation-execution/SKILL.md` (we are replacing it; keep the model-selection table, branch-safety section, four-status handling, and final whole-branch reviewer step from the existing version).

**Mode:** port

**Adaptation notes (from spec §3 row 1):**

- Process diagram: collapse `.t`+`.i` into a single per-task block.
- Branch safety: keep as top-level section, refuses `main`/`master`.
- Integration: code-distilling skills only (`analyzing-reference`, `distillation-design`, `distillation-plan`, `equivalence-tdd`). Drop all `superpowers:` references.
- Example workflow: rewrite in porting vocabulary (copy/port/learn-then-rewrite chunk example with implementer dispatch → spec-reviewer APPROVED → code-quality reviewer ISSUES → re-dispatch → APPROVED → final whole-branch reviewer).
- Model-selection table: preserve current code-distilling table.
- "Continuous execution" stance: preserve verbatim.
- Four-status handling (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED): preserve.
- Final whole-branch reviewer dispatch: preserve as terminating step.
- Plan path convention: `docs/plans/<date>-distill-<repo>-<feature>.md`.

**Equivalence test (spec §7 items 1–8):**

1. Per-task block in the process diagram dispatches ONE implementer (no `.t`/`.i`).
2. Spec-compliance reviewer runs BEFORE code-quality reviewer.
3. **Branch Safety** is a top-level section refusing `main`/`master`.
4. Model-selection table includes rows for copy / port / cross-language port / learn-then-rewrite / spec reviewer / code-quality reviewer / final reviewer.
5. Four-status handling block present and semantically unchanged.
6. Integration section names only code-distilling skills.
7. Final whole-branch reviewer dispatch is the terminating step.
8. "Continuous execution" stance preserved.

- [ ] **Step 1:** Read `<REF_PATH>/skills/subagent-driven-development/SKILL.md` for shape, tone, process-diagram structure, model-selection prose.
- [ ] **Step 2:** Read existing `skills/distillation-execution/SKILL.md` for content to preserve (branch-safety section, model-selection table, four-status handling, final whole-branch reviewer, example commit-message conventions).
- [ ] **Step 3:** Write the new SKILL.md to `skills/distillation-execution/SKILL.md`, merging SDD's shape with code-distilling's preserved content. Apply every adaptation note above.
- [ ] **Step 4:** Walk the spec §7 checklist items 1–8 against the written file. If any item fails, fix the file. Do not proceed to commit until all 8 pass.
- [ ] **Step 5:** Commit.

```bash
git add skills/distillation-execution/SKILL.md
git commit -m "distill(superpowers/sdd): port SKILL.md — collapse per-chunk tasks"
```

---

## Task 2: Port `implementer-prompt.md`

**Files:**
- Modify: `skills/distillation-execution/implementer-prompt.md`
- Read: `<REF_PATH>/skills/subagent-driven-development/implementer-prompt.md`
- Read for context: existing `skills/distillation-execution/implementer-prompt.md` (preserve Curated Context block, hard rules, mode-specific equivalence-tdd discipline, four-status DONE template).

**Mode:** port

**Adaptation notes (from spec §3 row 2):**

- Use SDD's section ordering and conversational tone as scaffolding.
- Inject the Curated Context block: spec row table, reference-map excerpt, source file content (for copy/port — OMITTED for learn-then-rewrite), exact test command, fail-state and pass-state expectations, working directory.
- Replace SDD's soft "Did I follow TDD if required?" with MANDATORY equivalence-tdd: port test → run failing → port impl → run passing → single commit.
- Keep the Hard Rules block: do NOT modify the reference path, do NOT read the reference repo from the subagent, do NOT weaken the test, do NOT skip the failing-state run, do NOT touch files outside the named list, do NOT paste reference lines for learn-then-rewrite.
- Keep the four-status report block (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED) including the DONE template's commit SHA, files list, and failing→passing test output lines.
- Keep mode-specific copy/port/learn-then-rewrite definitions inside the equivalence-tdd section.

**Equivalence test (spec §7 items 9–14):**

9. Curated Context block names every required field (spec row, ref-map excerpt, source content (or omission), test command, fail/pass expectations, working dir).
10. Required Sub-Skill section names `code-distilling:equivalence-tdd` and enumerates its 5 steps.
11. Hard Rules block forbids the six items listed in spec §7 item 11.
12. Self-Review section asks about equivalence (fail/pass), mode adherence, and scope.
13. Report Format specifies all four statuses; DONE template includes commit SHA, files, and failing→passing test output lines.
14. Tone matches SDD (conversational, "always OK to escalate"), not a wall of rules.

- [ ] **Step 1:** Read `<REF_PATH>/skills/subagent-driven-development/implementer-prompt.md` for shape, conversational tone, self-review structure, four-status semantics.
- [ ] **Step 2:** Read existing `skills/distillation-execution/implementer-prompt.md` for Curated Context block structure, hard rules block, equivalence-tdd 5 steps, mode-specific definitions, four-status DONE template.
- [ ] **Step 3:** Write the new implementer-prompt.md, using SDD's shape and tone as scaffolding while injecting all of code-distilling's porting-specific content. Target ≤ 160 lines (current is 182; SDD is 113).
- [ ] **Step 4:** Walk the spec §7 checklist items 9–14 against the written file. Fix gaps before commit.
- [ ] **Step 5:** Commit.

```bash
git add skills/distillation-execution/implementer-prompt.md
git commit -m "distill(superpowers/sdd): port implementer-prompt.md — SDD shape + equivalence-tdd"
```

---

## Task 3: Port `spec-reviewer-prompt.md`

**Files:**
- Modify: `skills/distillation-execution/spec-reviewer-prompt.md`
- Read: `<REF_PATH>/skills/subagent-driven-development/spec-reviewer-prompt.md`
- Read for context: existing `skills/distillation-execution/spec-reviewer-prompt.md` (for the controller's parse-compatible APPROVED/ISSUES output format).

**Mode:** copy (adaptations are mechanical)

**Adaptation notes (from spec §3 row 3):**

- Replace SDD's "Task tool (general-purpose)" preamble with a direct prompt body (matches code-distilling's existing reviewer-prompt style).
- Replace SDD's `[FULL TEXT of task requirements]` block with the spec row (Source chunk / Target file(s) / Mode / Deciding criterion / Adaptation notes).
- Replace SDD's `[From implementer's report]` with `**Implementer commit SHA:** <SHA>` and `**Files touched by the commit:** <LIST>`.
- Preserve verbatim: the "Do Not Trust the Report" stance, the three buckets (missing requirements / extra-unneeded work / misunderstandings), "Verify by reading code, not by trusting report".
- Report format: `APPROVED` / `ISSUES` (with one-per-line required fixes), parser-compatible with the controller.

**Equivalence test (spec §7 items 15–18):**

15. Three buckets present: missing requirements, extra/unneeded work, misunderstandings.
16. "Do Not Trust the Report" stance verbatim: read code at the SHA, not the implementer's narrative.
17. Header takes Task ID, implementer commit SHA, the spec row, files touched.
18. Report format: `APPROVED` / `ISSUES`, one-per-line required fixes.

- [ ] **Step 1:** Read `<REF_PATH>/skills/subagent-driven-development/spec-reviewer-prompt.md` for the three-bucket prose, the "do not trust" stance, the "verify by reading code" instruction.
- [ ] **Step 2:** Read existing `skills/distillation-execution/spec-reviewer-prompt.md` for the APPROVED/ISSUES output template and spec-row header layout.
- [ ] **Step 3:** Write the new spec-reviewer-prompt.md by copying SDD's body verbatim where it serves, swapping the header for the spec-row form, and pinning the output format to APPROVED/ISSUES. Target ≤ 80 lines (current is 101; SDD is 61).
- [ ] **Step 4:** Walk the spec §7 checklist items 15–18 against the written file. Fix gaps before commit.
- [ ] **Step 5:** Commit.

```bash
git add skills/distillation-execution/spec-reviewer-prompt.md
git commit -m "distill(superpowers/sdd): port spec-reviewer-prompt.md — adopt 3-bucket form"
```

---

## Task 4: Rewrite `code-quality-reviewer-prompt.md`

**Files:**
- Create or modify: `skills/distillation-execution/code-quality-reviewer-prompt.md`
- Read for structural template: existing `skills/distillation-execution/code-quality-reviewer-prompt.md` (use as the structural sibling for layout / parsing).
- **DO NOT read** `<REF_PATH>/skills/subagent-driven-development/code-quality-reviewer-prompt.md` for content beyond the high-level "Strengths / Issues / Assessment" output shape and the four extra check bullets. The reference is a 25-line pointer at a superpowers-only template that does not exist in target; this is a learn-then-rewrite chunk.

**Mode:** learn-then-rewrite

**Adaptation notes (from spec §3 row 4):**

Write from the behavioral checklist below; do NOT paste lines from `<REF_PATH>/skills/requesting-code-review/code-reviewer.md`. The reviewer:

- Header takes Task ID, implementer commit SHA, files touched, mode (copy / port / learn-then-rewrite).
- Instructs the reviewer to read target project conventions (style guide, test framework, sibling files) before reviewing.
- Reads code at the SHA, not the implementer's narrative.
- Reports: Strengths / Issues (Critical / Important / Minor) / Assessment.
- Extra porting-specific checks: single-responsibility per file, plan-conformant file layout, file-size growth flagged, mode-appropriate idioms (copy is minimal-diff, port is structure-preserving, learn-then-rewrite reads as native target code).
- Output format: `APPROVED` / `ISSUES` matching the spec-reviewer's format.

**Equivalence test (spec §7 items 19–23):**

19. Header takes Task ID, implementer commit SHA, files touched, mode.
20. Instructs reviewer to read target project conventions before reviewing.
21. Reviewer reads code at the SHA, not the implementer's narrative.
22. Output sections: Strengths / Issues (Critical / Important / Minor) / Assessment.
23. Extra porting-specific checks: single-responsibility, plan-conformant layout, file-size growth, mode-appropriate idioms.

- [ ] **Step 1:** Read the existing `skills/distillation-execution/code-quality-reviewer-prompt.md` for structural sibling layout and APPROVED/ISSUES wrapping.
- [ ] **Step 2:** Write a fresh code-quality-reviewer-prompt.md from the behavioral checklist above. Do NOT pull lines from the superpowers reference — this is a learn-then-rewrite chunk. Target ≤ 100 lines (current is 116; SDD is 25 but is a pointer).
- [ ] **Step 3:** Walk the spec §7 checklist items 19–23 against the written file. Fix gaps before commit.
- [ ] **Step 4:** Commit.

```bash
git add skills/distillation-execution/code-quality-reviewer-prompt.md
git commit -m "distill(superpowers/sdd): rewrite code-quality-reviewer-prompt.md (learn-then-rewrite)"
```

---

## Task 5: Walk the synthetic 2-chunk dry-run (spec §7)

**Files:**
- Modify: `docs/distilling/superpowers-subagent-driven-development-dry-run.md` (new file — the trace record).
- Read: the four ported files from tasks 1–4.

**Mode:** verification (not a port — this task is the equivalence check)

**Procedure:**

Author the synthetic 2-chunk plan fixture (from spec §7 dry-run section) inside the trace document, then walk the new `distillation-execution` skill against it mentally and record what dispatches the controller would make. The trace must include:

- Chunk A (`copy` mode, port a small TypeScript utility, e.g., `src/util/clamp.ts`, 30 lines): which prompt the controller would build for the implementer, with the pasted source.
- Chunk B (`learn-then-rewrite` mode, rewrite a reference's event-bus glue): which prompt the controller would build, with NO pasted reference source.
- Per chunk: implementer dispatch → spec-reviewer dispatch (APPROVED) → code-quality reviewer dispatch (APPROVED) → mark complete.
- After both chunks: final whole-branch reviewer dispatch.
- Confirm: exactly 2 implementer dispatches (NOT 4, i.e. no `.t`/`.i` split); chunk A prompt contains pasted source; chunk B prompt does NOT; no "should I continue?" between chunks.

**Equivalence test:** the trace matches every bullet in spec §7's dry-run section.

- [ ] **Step 1:** Create `docs/distilling/superpowers-subagent-driven-development-dry-run.md`. Include the synthetic 2-chunk fixture and the walked trace.
- [ ] **Step 2:** Validate the trace against spec §7's six dry-run bullets. If any bullet fails, the corresponding ported file (most likely SKILL.md) has a gap — fix it, then re-run this task.
- [ ] **Step 3:** Commit.

```bash
git add docs/distilling/superpowers-subagent-driven-development-dry-run.md
git commit -m "distill(superpowers/sdd): record dry-run trace for equivalence check"
```

---

## Final whole-branch review

After Tasks 1–5 are all committed, dispatch (or have the human reviewer perform) a whole-branch code review of the diff from the branch base. Confirm:

- Spec §9 success criteria all met (23 checklist items satisfied; dry-run trace matches; hidden coupling resolved; upstream skills still resolve `distillation-execution`; branch safety preserved; final reviewer dispatch preserved).
- No unrelated files touched.

If issues: amend the relevant chunk's task and re-execute that task only.
