---
name: using-code-distilling
description: Use when the user wants to port, distill, copy, borrow, or bring a feature from a reference repository into their project, or to catch an existing port up with its reference or close its remaining gaps.
---

# Using Code Distilling

Port the decisions that make a reference feature work — algorithms, architecture, prompts, workflow, domain rules, edge cases — re-expressed in the target's conventions, preserving the behavior that matters.

Scope: reference-based feature adoption, including keeping an existing port current. Not unrelated debugging, refactoring, or original development. Establish reference and target from the request and workspace; **ask for a missing reference rather than inventing one.**

## Small ports: implement directly

A port is small only when **all** of these hold:

- The reference behavior is a bounded unit you can read completely: a function, helper, regex, constant table, validation rule, small component, or localized fix. Workflows, multi-component features, stateful or concurrent logic, and prompt-driven or agent behavior are not small.
- It lands in one place in the target through existing interfaces — no new dependencies, no architectural change.
- No design question: scope is clear, and no integration difference changes observable behavior.

Then skip the design conversation, spec document, and review gate:

1. Read the reference unit, the dependencies that determine its behavior (defaults, constants, tests), and where it lands in the target.
2. Say in one line that this is a small port you are implementing directly. Do it without waiting.
3. Re-express it in the target's conventions, preserving the values and invariants its behavior depends on. Keep required license and attribution notices.
4. Run a check that would catch a wrong port. Report: reference path and revision, what was preserved or adapted, the checks, any limits.

If a design question appears — a scope choice, a behavior-changing integration difference, a new dependency — **stop and ask** before continuing. Size alone is not a design question: more code than expected stays small as long as you can still read it completely and nothing above changed. No longer small → switch to the full flow. User asks for a spec or a review → full flow regardless of size.

## Spec → review → implementation

1. Read [distillation-spec](../distillation-spec/SKILL.md). Inspect reference and target, explain the design to the user, ask the questions that shape the port, write `docs/code-distilling/<capability>/distillation-spec.md`.
2. **Stop for review.** Present the spec with your recommendations and open questions; wait for explicit approval. On feedback, revise and present again.
3. Read [distillation-implementation](../distillation-implementation/SKILL.md). Implement from the approved spec and the source, verify fidelity and integration, report.

<HARD-GATE>
Do not write port code or start implementation until the user has reviewed and explicitly approved the spec. A request to port the feature, even "end to end", is not approval of a spec the user has not seen. Only an explicit instruction to skip spec review waives this gate. This applies to every port that is not small under the criteria above; a port with any design question is not small.
</HARD-GATE>

- No separate plan stage, task document, or prewritten implementation code. The spec records durable decisions and evidence; the implementer chooses files, sequence, and tactics. A working checklist is optional.
- Analysis-only or spec-only request → deliver that and stop.
- Ask whenever a choice changes scope or observable behavior; routine implementation decisions are yours.
- While waiting for an answer: keep investigating, do not edit target code.

## Existing ports: close the gap

A port exists and the reference has moved, or the port falls short of its spec or reference → read [distillation-gap](../distillation-gap/SKILL.md), not a new spec. It inventories gaps against the recorded spec and revision, routes spec-changing decisions through the same review gate, and closes approved gaps with distillation-implementation.

## Harness portability

Use the host's skill loader if available; otherwise read the linked `SKILL.md` files directly. Paths are relative to their containing file.
