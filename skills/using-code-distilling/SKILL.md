---
name: using-code-distilling
description: Use when the user wants to port, distill, copy, borrow, or bring a feature from a reference repository into their project.
---

# Using Code Distilling

Port the decisions that make a reference feature work: algorithms, architecture, prompts, workflow, domain rules, and edge cases. Re-express them in the target project's conventions while preserving the behavior that matters.

Use this workflow for reference-based feature adoption, not unrelated debugging, refactoring, or original development. Establish the reference and target from the request and workspace; ask for a missing reference rather than inventing one.

## Spec → implementation

1. Read [distillation-spec](../distillation-spec/SKILL.md). Inspect the reference and target, then write `docs/code-distilling/<capability>/distillation-spec.md`: the behavioral contract and the design knowledge needed to reproduce it.
2. Read [distillation-implementation](../distillation-implementation/SKILL.md). Implement directly from that spec and the source, verify fidelity and integration, and report the result.

There is no separate plan stage, task document, or requirement to prewrite implementation code. The spec records durable decisions and evidence; the implementer chooses files, sequence, and local tactics as work develops. A working checklist is optional.

If the user requested an end-to-end port, continue after writing the spec. If they requested only analysis or a spec, deliver that and stop. Respect requested review gates and existing authorization. Ask when a missing choice changes scope or observable behavior, not for routine implementation decisions. Continue independent work while a decision is pending.

## Scale to the feature

A small utility may need a paragraph with source, invariants, exact values, target integration, and checks. A workflow feature needs enough architecture and domain context that another agent can reproduce its decisions without the original conversation. Add linked design notes or exact assets only when useful; do not manufacture empty sections.

For AI features, inspect prompts and the code that assembles context, selects tools or speakers, routes results, retries, and stops. A similar UI or a copied system prompt alone does not establish feature fidelity.

## Harness portability

Use the host's skill-loading mechanism if available; otherwise read the linked `SKILL.md` files directly. Paths are relative to their containing file. The workflow requires source inspection, file editing, and suitable verification, not a particular Skill, Task, Todo, or review tool. Work in the current session by default. Delegation is optional when available and authorized.

Existing `distillation-plan.md` artifacts may contain useful decisions. Reconcile those with the current spec and code when resuming an old port; do not require or regenerate a plan.
