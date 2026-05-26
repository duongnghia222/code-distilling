# Reference Map: superpowers — subagent-driven-development skill

**Reference path:** /Users/duongnghia/Desktop/nghia/superpowers
**Reference repo URL:** https://github.com/obra/superpowers.git
**Source commit hash:** f2cbfbefebbfef77321e4c9abc9e949826bea9d7
**Primary language:** Markdown (skill prompts) — no executable code in the feature
**Test framework:** none (skills are prose; no automated test suite for skill text)

## Feature scope (user's words)

Port the `subagent-driven-development` skill from `superpowers` and use it to **replace** the existing `code-distilling:distillation-execution` skill. Goal is to inherit SDD's lighter-weight execution model (one task per work item, smaller prompts, simpler spec reviewer) while keeping the porting-specific discipline that code-distilling needs (equivalence-tdd, mode-aware dispatch, source-file curation). The replacement must remain a code-distilling skill — i.e., it still consumes a reference map + spec + plan and dispatches a final whole-branch reviewer.

## Feature files

Paths in this section are RELATIVE to `<REF_PATH>`. Downstream skills join them with the `Reference path` field above to read each file.

- skills/subagent-driven-development/SKILL.md — the skill itself: when to use, process diagram, model-selection table, status handling, red flags, integration list. 279 lines.
- skills/subagent-driven-development/implementer-prompt.md — template for dispatching the implementer subagent: task text, context, before-you-begin questions, code-organization guidance, self-review, report format with DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT statuses. 113 lines.
- skills/subagent-driven-development/spec-reviewer-prompt.md — template for the spec compliance reviewer: "do not trust the report", three buckets (missing / extra / misunderstood), verify by reading code not by trusting report. 61 lines.
- skills/subagent-driven-development/code-quality-reviewer-prompt.md — template that delegates to `requesting-code-review/code-reviewer.md` with extra structure/file-size checks. 25 lines (very thin — it's a pointer).

## Transitive deps inside the repo

| File (relative to <REF_PATH>) | Used by | must-take or can-stub? | Why |
| ----------------------------- | ------- | ---------------------- | --- |
| skills/requesting-code-review/code-reviewer.md | skills/subagent-driven-development/code-quality-reviewer-prompt.md:11 | **can-stub** | Code-distilling already inlines its own 116-line `code-quality-reviewer-prompt.md`. The port should keep that inline pattern (code-distilling convention) rather than introducing a separate `requesting-code-review` skill. |
| skills/test-driven-development/SKILL.md | skills/subagent-driven-development/SKILL.md:276 ("Subagents follow TDD for each task") | **can-stub** (substitute) | Code-distilling already has `code-distilling:equivalence-tdd`, the porting-specific analog. The port must reference `equivalence-tdd`, not generic TDD. |
| skills/using-git-worktrees/SKILL.md | skills/subagent-driven-development/SKILL.md:270 ("Required workflow skills") | **can-stub** | Code-distilling does not have its own worktree skill and does not require worktrees. Drop this reference or replace with branch-safety check that already exists in `distillation-execution` (refuse to run on `main`/`master`). |
| skills/writing-plans/SKILL.md | skills/subagent-driven-development/SKILL.md:271 | **can-stub** (substitute) | Code-distilling has `code-distilling:distillation-plan` as the upstream artifact producer. The port must reference that skill, not `writing-plans`. |
| skills/finishing-a-development-branch/SKILL.md | skills/subagent-driven-development/SKILL.md:85, 273 (terminal step in the process diagram) | **can-stub** | Code-distilling does not have a branch-finishing skill. The current `distillation-execution` ends with a final whole-branch reviewer dispatch + announce-completion; the port should preserve that termination instead of pointing at a superpowers-only skill. |
| skills/executing-plans/SKILL.md | skills/subagent-driven-development/SKILL.md:279 (alternative workflow note) | **can-stub** | Out of scope. The "alternative workflow" callout can simply be dropped — code-distilling's flow assumes same-session execution. |

## External libraries

| Library | Version constraint | Used for | Equivalent in target? |
| ------- | ------------------ | -------- | --------------------- |
| Claude Code `Task` tool (general-purpose subagent) | n/a | Dispatching implementer + reviewer subagents | Identical — code-distilling already uses the same harness's `Task` tool (see `distillation-execution/SKILL.md:97`). |
| Claude Code `TodoWrite` / `TaskCreate` | n/a | Per-task tracking | Identical — code-distilling uses the same mechanism. |
| git | n/a | Commits, SHAs for review, branch safety | Identical. |

No third-party libraries — the feature is pure skill prose + harness tool usage.

## Tests for the feature

None. The reference skill has no automated tests. Its correctness is established by:

1. Human review of the prose for clarity and rigidity in the right places.
2. Behavioral observation when the skill is exercised end-to-end on a real task.

This is significant for the port: there is no "ported test" artifact to drive `equivalence-tdd`. **The equivalence test for this port will have to be a hand-authored behavioral check** — e.g., a dry-run dispatch on a synthetic 2-chunk plan, asserting that the new skill produces the same artifact shape (commit per task, two reviewer passes, final whole-branch review) as the SDD process diagram. This will be settled in `distillation-design` §7.

## Entry points

How callers invoke the feature from outside:

- **User-facing trigger** — user invokes `superpowers:subagent-driven-development` (in target: a `code-distilling` skill name TBD by design) after a written implementation plan exists. SDD's description: "Use when executing implementation plans with independent tasks in the current session." Code-distilling's analog must trigger on: "approved distillation plan exists and the user wants to execute it in this session."
- **Process entrypoint inside the skill** — SKILL.md `## The Process` digraph's first node: "Read plan, extract all tasks with full text, note context, create TodoWrite". This is the single entry to the per-task loop.
- **Subagent dispatch entrypoints** — three prompt templates dispatched via the `Task` tool: implementer, spec reviewer, code quality reviewer. Each is its own callable surface.

## Hidden coupling

(Reference path: /Users/duongnghia/Desktop/nghia/superpowers — paths below are relative.)

- skills/subagent-driven-development/SKILL.md:131 — example plan path is hardcoded as `docs/superpowers/plans/feature-plan.md`. Port must use code-distilling's plan path convention (`docs/plans/<date>-distill-<repo>-<feature>.md`, per existing `distillation-execution/SKILL.md:190`).
- skills/subagent-driven-development/code-quality-reviewer-prompt.md:11 — points at the file path `requesting-code-review/code-reviewer.md`, which assumes the superpowers skill layout. In the target, this template is inlined.
- skills/subagent-driven-development/SKILL.md:268-279 — entire "Integration" section names superpowers-specific sibling skills. Port must rewrite this section against code-distilling's skill graph (`analyzing-reference`, `distillation-design`, `distillation-plan`, `equivalence-tdd`).
- skills/subagent-driven-development/SKILL.md:239 — branch-safety phrasing ("Start implementation on main/master branch without explicit user consent") matches code-distilling's existing rule but is stated as a red-flag, not a gated step. Code-distilling's current `distillation-execution/SKILL.md:38-40` makes branch-safety a separate top-level section. Port should preserve code-distilling's stronger gating.
- skills/subagent-driven-development/SKILL.md:138-148 — Task 1 example uses superpowers domain vocabulary ("hook installation script", "~/.config/superpowers/hooks/"). Port must rewrite the example with porting-domain vocabulary (a copy/port/learn-then-rewrite task on a chunk).
- skills/subagent-driven-development/implementer-prompt.md:90 — "Did I follow TDD if required?" — soft TDD. In the porting context, equivalence-tdd is **mandatory per task**. Port must harden this to "Did I follow equivalence-tdd?" with no "if required" softness.
- No env vars, no file IO (beyond reading the plan), no network, no time/random sources, no globals. The skill is stateless prose; coupling is entirely textual/conventional.

## Open questions for design

1. **Two-task split (`.t` + `.i`) — keep or drop?** Existing `distillation-execution` enforces test task + impl task per chunk; SDD has one task per item. The user identified the 2× multiplication as a major slowness cause. Design must decide: collapse to one task per chunk (matching SDD), keep the split only for `learn-then-rewrite` mode, or keep it always. Affects every downstream prompt.
2. **Source-file paste — keep, drop, or mode-gated?** SDD never pastes source from a reference; code-distilling's current `implementer-prompt.md:33-35` pastes the full source file for copy/port. This is genuinely useful for porting (the subagent literally cannot read the reference repo), but it inflates every dispatch. Design must pick: always paste, paste only for `copy`, or summarize-then-link.
3. **Spec reviewer rigor.** SDD's spec reviewer is 61 lines ("missing / extra / misunderstood"); code-distilling's is 101 lines with 5 ordered §-by-§ checks. The 5-check reviewer is more likely to bounce work back into re-review loops. Design must pick a target rigor — the simpler SDD form may miss adaptation-note drift.
4. **Branch-safety positioning.** Keep it as a top-level gating section (current code-distilling style, stronger) vs. a red-flag bullet (SDD style, weaker)? Recommend keeping the stronger form, but the design should call this out explicitly so it isn't lost during the port.
5. **Equivalence test for the port itself.** There is no automated reference test. Design (§7) must specify what counts as "the new skill behaves equivalently to a curated subset of SDD on a synthetic distillation plan" — e.g., a numbered behavioral checklist that a reviewer can walk through against a dry-run.
6. **Skill name in the target.** Replace `distillation-execution` with the same name (overwriting the skill), or rename (e.g., `executing-distillation`) and update `using-code-distilling` references? The user said "replace" — recommend keeping the name `distillation-execution` so the upstream `using-code-distilling` flow and red-flag references continue to point at the right skill.
