---
name: using-code-distilling
description: Use when the user wants to port, distill, copy, borrow, or "bring in" a feature or capability from a reference open-source repo into their own project. Establishes the 5-stage distillation flow and its human gates.
---

# Using Code Distilling

Distillation takes the *encoded decisions* from a reference implementation — the algorithm, the tuned constants, the hard-won edge cases — and re-expresses them in your project. Keep the gold, drop their packaging.

**The code is not the asset. The decisions encoded in the code are the asset.**

This plugin is **not** a general coding workflow. It engages only when porting intent is present. For everything else, fall back to your default workflow.

## When to Engage

Engage when ANY of these is true:

- The user wants to port / distill / copy / borrow / "bring in" a feature from another repo.
- The user hands you a path or URL to a reference repo and wants to adopt code from it.
- The user says "there's a good implementation of X over there, let's use it."

Do **not** engage when:

- The user is writing original code from scratch with no reference repo.
- The user is debugging or refactoring their own existing code.
- The work is general software engineering unrelated to porting.

If porting intent is present but no reference path is supplied yet, ask for one before starting Stage 1. Don't guess where the reference lives.

## The Flow

Five stages, with human gates after the analysis, the spec, and the plan. **Each stage writes a doc** — because the implementation stage dispatches isolated-context subagents, the docs are how the stages talk to each other. A subagent that can't see your conversation can read the plan.

```dot
digraph distilling {
    "Reference + porting intent?" [shape=diamond];
    "Stage 1: reference-analysis" [shape=box];
    "Gate: capability + copy list ok?" [shape=diamond];
    "Stage 2: distillation-spec" [shape=box];
    "Gate: keep/discard + verify plan ok?" [shape=diamond];
    "Stage 3: distillation-plan" [shape=box];
    "Gate: task breakdown ok?" [shape=diamond];
    "Stage 4: distillation-implementation" [shape=box];
    "Stage 5: gap-report (verify)" [shape=box];
    "Gate: gaps acceptable?" [shape=diamond];
    "Done" [shape=doublecircle];
    "Fall back to default workflow" [shape=doublecircle];

    "Reference + porting intent?" -> "Stage 1: reference-analysis" [label="yes"];
    "Reference + porting intent?" -> "Fall back to default workflow" [label="no"];
    "Stage 1: reference-analysis" -> "Gate: capability + copy list ok?";
    "Gate: capability + copy list ok?" -> "Stage 1: reference-analysis" [label="no, refine"];
    "Gate: capability + copy list ok?" -> "Stage 2: distillation-spec" [label="yes"];
    "Stage 2: distillation-spec" -> "Gate: keep/discard + verify plan ok?";
    "Gate: keep/discard + verify plan ok?" -> "Stage 2: distillation-spec" [label="no, refine"];
    "Gate: keep/discard + verify plan ok?" -> "Stage 3: distillation-plan" [label="yes"];
    "Stage 3: distillation-plan" -> "Gate: task breakdown ok?";
    "Gate: task breakdown ok?" -> "Stage 3: distillation-plan" [label="no, refine"];
    "Gate: task breakdown ok?" -> "Stage 4: distillation-implementation" [label="yes"];
    "Stage 4: distillation-implementation" -> "Stage 5: gap-report (verify)";
    "Stage 5: gap-report (verify)" -> "Gate: gaps acceptable?";
    "Gate: gaps acceptable?" -> "Stage 4: distillation-implementation" [label="no, fix"];
    "Gate: gaps acceptable?" -> "Done" [label="yes"];
}
```

## The Stages

| Stage | Skill | Produces | Gate |
|-------|-------|----------|------|
| 1 | `reference-analysis` | `reference-analysis.md` — map the core, explain their design, agree what to copy | human: capability + copy list |
| 2 | `distillation-spec` | `distillation-spec.md` — contract · keep-verbatim · discard · seam→your-deps · per-chunk modes · verify plan | human: keep/discard + verify plan |
| 3 | `distillation-plan` | `distillation-plan.md` — source→target file map · bite-sized tasks with code, keep-verbatim, seams | human: task breakdown |
| 4 | `distillation-implementation` | the code + commits — execute the plan | none (runs continuously) |
| 5 | `gap-report` | `gap-report.md` — completeness · fidelity · no-leakage · contract | human: skim + accept |

Artifacts live in `docs/code-distilling/<capability>/`.

## Human Judgment

Gates are **between stages** (1→2, 2→3, 3→4, and 5→done). Within Stage 4, subagents run **continuously** without per-task check-ins — that's by design and is not a missing gate.

## It Scales Down

A one-file copy gets a few-sentence analysis, a one-paragraph spec, a two-task plan, a quick implementation, and a fast gap-check. **The docs shrink; the stages stay.** Skipping a stage is not faster — it's how a distillation turns into untraceable paste.
