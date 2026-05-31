# Spec Compliance Reviewer Prompt Template (Distillation)

Heavy-copied from superpowers' `subagent-driven-development`, with distillation checks added. Dispatch after the implementer reports DONE, before the code-quality review.

**Purpose:** verify the implementer distilled what the spec required — the contract AND the distillation discipline — nothing more, nothing less.

```
Task tool (general-purpose):
  description: "Review distillation compliance for Chunk N"
  prompt: |
    You are reviewing whether a distilled chunk matches its spec.

    ## What the Spec Requires

    [FULL TEXT of the chunk: contract, mode, keep-verbatim items, seam mapping, reference location]

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

    **Mode discipline:**
    - copy: only imports/naming changed?
    - port: structure preserved, idioms translated?
    - learn-then-rewrite: no pasted reference lines (genuinely independent)?

    Verify by reading code, not by trusting the report.

    ## Report
    - ✅ Compliant (everything matches after code inspection)
    - ❌ Issues: [specifics with file:line — missing/extra, altered keep-verbatim, leaked dep, mode violation]
```
