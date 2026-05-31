# Code Quality Reviewer Prompt Template (Distillation)

Adapted from superpowers' `subagent-driven-development`, made self-contained (this plugin doesn't ship `requesting-code-review`). Dispatch only after the spec-compliance review passes.

**Purpose:** verify the distilled chunk is well-built — idiomatic in the target, clean, tested, free of leaked reference cruft.

```
Task tool (general-purpose):
  description: "Code quality review for Chunk N"
  prompt: |
    You are reviewing the quality of a distilled chunk. Review the diff between [BASE_SHA] and [HEAD_SHA].

    ## Context

    [chunk summary from the implementer's report; the chunk's mode]

    ## Review For:

    **Idiomatic in this project's language/stack:**
    - Does it read like native target code, or like transliterated source-language code?
    - Names match what things do; target conventions followed (casing, error handling, async style).

    **Structure:**
    - Does each file have one clear responsibility with a well-defined interface?
    - Can units be understood and tested independently?
    - Did this change create new large files or significantly grow existing ones? (Judge only what
      this change contributed; don't flag pre-existing sizes.)

    **Distillation cleanliness:**
    - Did accidental complexity from the reference leak in (its config, logging, telemetry,
      abstractions-for-their-scale)?
    - Are keep-verbatim constants/prompts isolated and clearly labeled, not scattered magic numbers?

    **Tests:**
    - Do tests verify behavior, not mocks? If equivalence-testing applied, are the tests reference-derived?

    ## Report
    - **Strengths**
    - **Issues** — Critical / Important / Minor, each with file:line
    - **Assessment** — approved | changes needed
```
