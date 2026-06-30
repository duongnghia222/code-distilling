---
name: distillation-plan
description: Stage 3 of code distilling. Use after the distillation spec is approved to turn it into a bite-sized, file-mapped implementation plan — a source→target file map and per-task steps carrying mode, keep-verbatim items, and seam substitutions, with complete code in every step. Produces distillation-plan.md.
---

# Distillation Plan (Stage 3)

## Overview

Turn the approved distillation spec into an implementation plan a zero-context engineer can execute. Document everything: the source (reference) → target file map, which files each task touches, the exact code, the keep-verbatim items to reproduce, the seam substitutions to make, and when to commit. Bite-sized tasks.

Assume the implementer is a skilled developer who knows almost nothing about this project, the reference, or what's worth preserving. Everything they need to keep the gold and avoid leaking the reference's deps must be in the plan.

<HARD-GATE>
Do NOT dispatch implementers or write any port code until the plan is written and the user has approved it. This applies to EVERY port regardless of how small it looks.
</HARD-GATE>

**Save plans to:** `docs/code-distilling/<capability>/distillation-plan.md`

## Input

The approved `distillation-spec.md`: the contract, the keep-verbatim list, the discard list, the seam→your-deps mapping, the per-chunk modes, and the verification strategy. Every task in the plan traces back to a chunk in the spec. If the spec is missing or unapproved, go back to `distillation-spec` first.

## Scope Check

If the spec covers multiple independent capabilities, it should have been split during the spec stage. If it wasn't, suggest splitting into separate plans — one per capability. Each plan should produce working code on its own.

## Source → Target File Map

Before defining tasks, map the reference's files to your project's files. This locks in decomposition.

| Reference (source) | Your project (target) | Mode | Notes |
|--------------------|-----------------------|------|-------|
| `ref/path/foo.py` | `src/path/foo.ts` (create) | port | seam: their store → your store |

Each target file should have one clear responsibility; files that change together live together. Follow the project's existing patterns — don't unilaterally restructure.

## Bite-Sized Task Granularity

Each step is one action (2–5 minutes). Every chunk follows the same shape: implement preserving keep-verbatim and wiring seams, spot-check, commit. The fidelity check happens in `gap-report`.

## Plan Document Header

Every plan MUST start with this header:

```markdown
# [Capability] Distillation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use code-distilling:distillation-implementation to execute this plan. It dispatches a fresh subagent per logic-heavy task (two-stage review) and implements simple tasks directly. Steps use checkbox (`- [ ]`) syntax.

**Goal:** [one sentence — what capability is being distilled into this project]

**Reference:** [repo @ commit] → **Target:** [where it lands]

**Approach:** [2–3 sentences — the modes in play, the main seams]

---
```

## Task Structure

Each task carries its distillation context, then bite-sized steps.

````markdown
### Task N: [Chunk Name]

**Files:**
- Source: `ref/path/to/source.py` (reference @ <commit>)
- Create: `src/exact/path.ts`

**Mode:** copy | port | learn-then-rewrite

**Keep-verbatim (reproduce EXACTLY — no rounding, rephrasing, reordering):**
- `THRESHOLD = 0.83` (ref source.py:42)
- [prompt template / step order / regex / lookup table …]

**Seam substitutions:**
- their `VectorStore` → this project's `src/db/store.ts`
- do NOT import the reference's framework/libraries

- [ ] **Step 1: Implement** — preserve keep-verbatim, wire seams

```ts
// complete code — for a port, keep-verbatim items appear verbatim here
```

- [ ] **Step 2: Spot-check / verify** — `<exact command>` — Expected: builds clean / pristine output

- [ ] **Step 3: Commit**

```bash
git add <files>
git commit -m "distill(<repo>): <what was distilled>"
```
````

## No Placeholders

Every step must contain the actual content. These are plan failures — never write them:

- "TBD", "TODO", "implement later", "port the rest"
- "Add error handling" / "handle edge cases" without the code
- "Preserve their constants" without listing the actual values
- "Similar to Task N" (repeat the code — tasks may be read out of order)
- A keep-verbatim item named but not shown (the implementer can't preserve what isn't there)
- References to types/functions not defined in any task

## Remember

- Exact file paths always — both source (reference) and target.
- Complete code in every code step. Keep-verbatim items appear verbatim in the task.
- Exact commands with expected output.
- Seam substitutions named per task; no reference deps introduced.
- DRY, YAGNI, frequent commits.

## Self-Review

After writing the plan, check it against the spec with fresh eyes (your own checklist, not a subagent dispatch):

1. **Chunk coverage:** does every chunk in the spec have a task? List gaps.
2. **Keep-verbatim coverage:** does every keep-verbatim item appear, verbatim, in some task? A constant in the spec but missing from the plan is a dropped trick.
3. **Placeholder scan:** any of the "No Placeholders" red flags? Fix them.
4. **Seam/mode consistency:** does each task's mode match the spec? Are seam substitutions specified? Does any task quietly import a reference dep?
5. **Type consistency:** do signatures/names used in later tasks match earlier ones?

Fix issues inline. For a large plan, optionally dispatch `./plan-document-reviewer-prompt.md` for an independent check.

## User Review Gate

> "Distillation plan written to `<path>`. Please review the task breakdown and the keep-verbatim items before we implement."

Wait for approval. On changes, update and re-run the self-review. Only proceed to `distillation-implementation` once the user approves.
