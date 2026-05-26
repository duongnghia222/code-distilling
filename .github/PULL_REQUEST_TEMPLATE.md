<!--
BEFORE SUBMITTING: Read every word of this template. PRs that leave
sections blank, contain multiple unrelated changes, or show no evidence
of human involvement will be closed without review.
-->

## What problem are you trying to solve?

<!-- Describe the specific problem you encountered. If this was a session
     issue (the agent got the porting flow wrong, license check failed
     incorrectly, equivalence-TDD discipline broke down), include: what you
     were doing, what went wrong, the model's exact failure mode, and
     ideally a transcript or session log.

     "Improving" something is not a problem statement. What broke? What
     failed? What was the user experience that motivated this? -->

## What does this PR change?

<!-- 1-3 sentences. What, not why — the "why" belongs above. -->

## Is this change appropriate for the core plugin?

<!-- `code-distilling` core contains the porting workflow: reference
     analysis, design, planning, execution, equivalence-TDD, and
     attribution. Ask yourself:

     - Would this be useful to someone porting code from a *completely
       different* reference repo than the one that motivated your change?
     - Is this project-specific, language-specific, or domain-specific?
     - Does this integrate or promote a third-party service?

     If your change is a new skill for a specific domain or third-party
     integration, it belongs in its own plugin — not here. -->

## What alternatives did you consider?

<!-- What other approaches did you try or evaluate before landing on this
     one? Why were they worse? If you didn't consider alternatives, say so
     — but know that's a red flag. -->

## Does this PR contain multiple unrelated changes?

<!-- If yes: stop. Split it into separate PRs. Bundled PRs will be closed.
     If you believe the changes are related, explain the dependency. -->

## Existing PRs

- [ ] I have reviewed all open AND closed PRs for duplicates or prior art
- Related PRs: <!-- #number, #number, or "none found" -->

<!-- If a related closed PR exists, explain what's different about your
     approach and why it should succeed where the other didn't. -->

## Environment tested

| Harness (e.g. Claude Code, Codex) | Harness version | Model | Model version/ID |
|------------------------------------|-----------------|-------|------------------|
|                                    |                 |       |                  |

## New harness support (required if this PR adds a new harness)

<!-- If this PR adds support for a new harness (IDE, CLI tool, agent
     runner), you MUST include a session transcript proving the
     integration actually works.

     A real integration loads the `using-code-distilling` bootstrap at
     session start. The bootstrap is what causes the porting skills to
     auto-trigger when porting intent is detected. Without it, the skills
     are dead weight — present on disk but never invoked at the right
     moments.

     ACCEPTANCE TEST: Open a clean session in the new harness inside any
     project, with a reference repo cloned anywhere on disk. Send exactly
     this user message (substitute a real path):

         Let's port the X feature from /path/to/reference-repo.

     A working integration auto-triggers `analyzing-reference` before any
     code is written. Paste the complete transcript below.

     These are NOT real integrations and PRs that ship them will be closed:

     - Manually copying skill files into the harness.
     - Wrapping with `npx skills` or similar at-runtime shims.
     - Anything that requires the user to opt in to skills per-session.
     - Anything where `analyzing-reference` does not auto-trigger on the
       test above.
-->

<details>
<summary>Clean-session transcript for the acceptance test</summary>

```
paste the complete transcript here
```

</details>

## Evaluation

- What was the initial prompt you (or your human partner) used to start
  the session that led to this change?
- How many eval sessions did you run AFTER making the change?
- How did outcomes change compared to before the change?

<!-- "It works" is not evaluation. Describe the before/after difference
     you observed across multiple sessions. Porting flows have a lot of
     branching — license outcomes, mode decisions, equivalence-TDD
     discipline. Demonstrate that your change improves outcomes on more
     than one path. -->

## Rigor

- [ ] If this is a skills change: I tested the change adversarially
      across multiple sessions and pasted results below.
- [ ] This change was tested against at least one real reference repo,
      not only on contrived examples.
- [ ] I did not modify carefully-tuned content (Red Flags tables,
      rationalizations, Iron Law statements, "your human partner"
      language) without evidence the change is an improvement.

<!-- If you changed wording in skills that shape agent behavior, show your
     eval methodology and results. These are not prose — they are code. -->

## Human review

- [ ] A human has reviewed the COMPLETE proposed diff before submission.

<!--
STOP. If the checkbox above is not checked, do not submit this PR.

PRs will be closed without review if they:
- Show no evidence of human involvement.
- Contain multiple unrelated changes.
- Promote or integrate third-party services or tools.
- Submit project-specific or personal configuration as core changes.
- Leave required sections blank or use placeholder text.
- Modify behavior-shaping content without eval evidence.
-->
