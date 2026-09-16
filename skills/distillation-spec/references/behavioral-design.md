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

Preserve the relationship between instructions and execution. A tool renamed in code must also change in its prompt/schema through a documented adaptation. Taking only a top-level prompt while omitting examples, context policy, or validation loses part of the design.

## Verify nondeterministic features

Use recorded or stubbed model/tool outputs to check routing, parsing, context assembly, state changes, and stopping deterministically. Check re-expressed assets against what the spec says they must do, separately. When live evaluation is available and authorized, check outcome quality on representative cases and record model/settings and limitations. Do not require identical generated prose, or claim quality parity from mocked control-flow checks alone.
