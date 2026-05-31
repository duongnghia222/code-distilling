# Gap Report Reviewer Prompt Template

Dispatch a fresh-eyes verifier (NOT the agent that wrote the code) to compare the implementation against the spec and the reference.

```
Task tool (general-purpose):
  description: "Gap-report verification for <capability>"
  prompt: |
    You are verifying whether a distillation captured its reference. You did not write this code.
    Trust nothing; verify by reading.

    ## Inputs
    - Distillation spec: [path]
    - Reference core: [paths]
    - Implementation: [paths]

    ## Your Job — produce a gap report on four checks:

    **Completeness:** For every keep-verbatim item and every chunk in the spec, find it in the
    implementation. List present (file:line) and missing.

    **Fidelity:** For each keep-verbatim item, compare the ported value to the reference value. Flag
    any altered: rounded constants, reworded prompts, reordered steps, changed regexes/tables. Show
    reference value vs ported value.

    **No-leakage:** Does the implementation import the reference's framework/libraries, or carry its
    accidental complexity (config, logging, telemetry, abstractions-for-their-scale)? Are seams
    wired to this project's deps per the spec?

    **Contract:** Does the behavior satisfy the spec's contract and invariants? Reason from the code;
    note where you'd want an executable test to be sure.

    ## Report Format
    - **Verdict:** FULLY DISTILLED | GAPS FOUND
    - **Completeness:** present / missing (file:line)
    - **Fidelity:** altered items (reference vs ported) or "all faithful"
    - **Leakage:** found deps/complexity or "none"
    - **Contract:** assessment + uncertainties
    - **Gaps:** concrete list, each naming the chunk to fix
```
