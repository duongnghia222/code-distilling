# Fidelity review

Use in the current session or as an authorized reviewer handoff. Supply the spec, relevant reference revision/locations, design assets, target diff, and check results.

Read actual source and target code; do not treat the implementer's report as evidence of correctness. Check:

- Does the implemented path satisfy the contract, including side effects and failure cases?
- Are exact assets unchanged in their required representation?
- Are the algorithm, critical ordering, state ownership, transition predicates, domain heuristics, and termination preserved?
- For AI features, do prompts, context assembly, tool schemas, parsing, and routing still work together as specified?
- Are seam differences resolved in behavior, rather than merely renaming calls?
- Are deviations explicit and within scope? Has discarded packaging leaked into the target, or essential behavior disappeared with it?
- Do checks distinguish this port from a plausible but incorrect imitation? What remains unverified?
- Are source provenance and required attribution preserved?

Report concrete issues with target locations, source/spec evidence, behavioral impact, and missing verification. Separate implementation defects from spec ambiguities. State whether fidelity is supported by available evidence, and name its limits.
