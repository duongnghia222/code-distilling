# Integration and code-quality review

Use in the current session or as an authorized reviewer handoff. Supply the target diff, spec integration decisions, and relevant check results. Review actual changed code and its callers.

Check correctness and maintainability in the target: clear responsibilities, native conventions, interface compatibility, error handling, resource lifetime, async behavior, and appropriate tests. Look for imported reference scaffolding, hidden external-checkout dependencies, duplicated retries/caches, and changes outside the capability.

Judge the change's contribution, not unrelated pre-existing problems. Do not recommend simplifying away behavior the spec requires; resolve fidelity issues using [spec-reviewer-prompt.md](spec-reviewer-prompt.md).

Report material issues with locations, impact, and supporting evidence; separate optional improvements from defects. State checks performed and limitations. A clean review does not substitute for an integrated behavior check.
