# Extracting behavioral design

Read this for features whose quality depends on prompts, domain rules, or multi-step agent workflows. These are inspection lenses, not a prescribed architecture. Record only mechanisms present in the reference or explicitly requested adaptations.

## Trace execution and branches

Follow a concrete input through the actual entry point. For each meaningful transition, capture the trigger, state read, decision rule, action, state written, and next/stop condition. Follow a failure or alternate path too. Cite the source symbol controlling each important transition.

Separate model decisions from deterministic controller decisions. A prompt asking an agent to stop differs from code enforcing a round limit. Document both when both exist. Note whether state is per request, per agent, per conversation, or shared; who may mutate it; and how delayed or duplicate events are handled.

## Prompts, context, and skills

Locate behavior-bearing assets, including templates assembled from multiple files. Record:

- Message roles/order, exact static text, examples, variable meanings, and dynamic insertion points.
- Context construction: selected history, memory, retrieved material, visibility boundaries, truncation/summarization, and token allocation.
- Tool names and schemas, availability by phase, result formatting, and errors that influence the next model action.
- Output format, parser/validator, repair/retry path, and controller action each valid output triggers.
- Model/settings where specified, fallback choices, and behavior depending on them.
- For skills: activation conditions, loading order, linked resources/scripts, and how results return to the workflow.

Preserve the relationship between instructions and execution. A tool renamed in code must also change in its prompt/schema through a documented adaptation. Copying only a top-level prompt while omitting examples, context policy, or validation loses part of the design.

## Example lens: deep research

Inspect whether and how the reference performs clarification, decomposition, query generation, source selection, fetching, extraction, evidence tracking, deduplication, gap assessment, follow-up search, synthesis, and citation binding. Identify absent mechanisms rather than inventing them.

Capture the loop's continuation rule and budget, how evidence remains attached to claims, how conflicting or inadequate evidence affects output, and what happens on failed retrieval. Inspect domain quality rules and prompt assets together with orchestration.

Useful checks might include duplicate sources, contradictory evidence, an unanswered subquestion, retrieval failure, and budget exhaustion. Derive expected outcomes from the source or an explicit target decision. A polished report alone cannot demonstrate workflow fidelity.

## Example lens: agent group chat

Find actual reply-selection logic: who is eligible, how mentions/direct replies affect eligibility, ranking/tie-breaking or model-based speaker selection, when an agent stays silent, and how consecutive replies are limited.

Trace visible history for each participant, tool-result routing, shared versus private state, concurrent replies, cancellation, and termination. Check whether an agent's own message can trigger another reply and how cycles or duplicate events are controlled.

Useful traces might include an explicit mention, no eligible speaker, tied candidates, a tool result, a repeated event, and a round limit. Do not replace a selector with round-robin or broadcast unless that adaptation is intended.

## Verify nondeterministic features

Use recorded or stubbed model/tool outputs to check routing, parsing, context assembly, state changes, and stopping deterministically. Compare exact assets separately. When live evaluation is available and authorized, check outcome quality on representative cases and record model/settings and limitations. Do not require identical generated prose, or claim quality parity from mocked control-flow checks alone.
