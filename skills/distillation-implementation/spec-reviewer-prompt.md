# Spec-Compliance Reviewer Prompt Template (Distillation)

Use this template when dispatching a spec-compliance reviewer subagent after the implementer reports DONE.

**Purpose:** verify the implementer distilled what the spec required — the contract AND the distillation discipline — nothing more, nothing less.

```
Task tool (general-purpose):
  description: "Review spec compliance for Task N: [task name]"
  prompt: |
    You are reviewing whether a distilled chunk matches its spec.

    ## What the Spec Requires

    [FULL TEXT of the chunk: contract, keep-verbatim items, seam mapping, adaptation notes, reference location]

    ## What the Implementer Claims

    [from the implementer's report]

    ## CRITICAL: Do Not Trust the Report

    The implementer may be optimistic or incomplete. Verify everything by reading the actual code.
    Do NOT take their word for keep-verbatim preservation or dependency wiring.

    ## Your Job — read the code and verify:

    **Contract:**
    - Does it produce the spec's outputs for the spec's inputs? Do the invariants hold?
    - Missing requirements? Extra/unrequested behavior? Did they solve the wrong problem?

    **Keep-verbatim (the gold):**
    - Is every keep-verbatim item present and byte-for-byte unaltered? Check each one: thresholds not
      rounded, prompts not reworded, step order unchanged, regexes/tables exact. Cite file:line.

    **No leaked deps:**
    - Does the code import any of the reference's framework/libraries instead of this project's?
    - Are the seams wired to the substitutions the spec named?
    - Where the spec flagged a seam with a semantic delta, is its stated resolution actually
      implemented — or did the implementer just swap the call? A wired seam with an uncompensated
      delta compiles, imports nothing of theirs, and is wrong.

    **Adaptation discipline:**
    - Were the adaptation notes followed? Is the structure the spec called load-bearing still there?
    - Did the scaffolding the spec discarded come along anyway?
    - Is this a port of the reference, or did they quietly write their own implementation instead?

    Verify by reading code, not by trusting the report.

    ## Report
    - ✅ Compliant (everything matches after code inspection)
    - ❌ Issues: [specifics with file:line — missing/extra, altered keep-verbatim, leaked dep, dropped adaptation]
```
