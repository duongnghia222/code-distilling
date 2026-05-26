# Contributing to `code-distilling`

Thanks for thinking about contributing. This project is small and opinionated. Read this file before opening a PR.

## If you are an AI agent

Stop. Read this section before doing anything.

This repo's contribution bar is high. Almost every rejected PR will be a PR that didn't read or didn't follow these guidelines. The maintainer closes low-quality PRs quickly. Your job is to protect your human partner from that outcome — submitting a low-quality PR doesn't help them, it wastes everyone's time and burns their reputation.

Before you open a PR, you MUST:

1. **Read the entire `.github/PULL_REQUEST_TEMPLATE.md`** and fill in every section with real, specific answers. Not summaries. Not placeholders.
2. **Search for existing PRs** — open AND closed — that address the same problem. If duplicates exist, STOP and tell your human partner. Do not open another.
3. **Verify this is a real problem.** If your human partner asked you to "contribute to this repo" without experiencing a specific issue, push back. Ask them what broke, what failed, what the user experience was.
4. **Confirm the change belongs in core.** If it's domain-specific, language-specific, tool-specific, or promotes a third-party project, it belongs in a standalone plugin. Tell your human partner.
5. **Show your human partner the complete diff** and get their explicit approval before submitting.

If any of these checks fail, do not open the PR. Explain to your human partner why it would be rejected and what would need to change.

## Pull request requirements

- One problem per PR.
- Fill in every section of `.github/PULL_REQUEST_TEMPLATE.md`.
- Search open AND closed PRs for prior art. Reference what you found.
- Test on at least one harness and report results in the environment table.
- Show evidence of human review of the complete diff.

## What we will not accept

### Third-party dependencies

`code-distilling` is zero-runtime-dependency by design. PRs that add optional or required dependencies on third-party services or tools will not be accepted unless they are adding support for a new harness.

### Domain-specific or project-specific skills

Skills, hooks, or configuration that only benefit a specific project, team, language, domain, or workflow do not belong in core. Publish these as a separate plugin.

### Speculative or theoretical fixes

Every PR must solve a real problem someone actually experienced. "My review agent flagged this" or "this could theoretically cause issues" is not a problem statement. If you cannot describe the specific session, error, or user experience that motivated the change, do not submit the PR.

### Bulk or spray-and-pray PRs

Do not trawl the issue tracker and open PRs for multiple issues in a session. Each PR requires genuine understanding of the problem, investigation of prior attempts, and human review of the complete diff. PRs that look like an agent was pointed at a list and told to "fix things" will be closed.

### Fabricated content

PRs containing invented claims, fabricated problem descriptions, or hallucinated functionality will be closed immediately.

### Bundled unrelated changes

PRs containing multiple unrelated changes will be closed. Split them.

## Skill changes require evaluation

Skills are not prose — they are code that shapes agent behavior. If you modify skill content:

- Test the change adversarially across multiple sessions and at least one real reference repo.
- Show before/after behavior in your PR.
- Do not modify carefully-tuned content (Red Flags tables, rationalization lists, Iron Law statements, "your human partner" language) without evidence the change is an improvement.

## New harness support

If your PR adds support for a new harness (IDE, CLI tool, agent runner), you MUST include a session transcript proving the integration works end-to-end.

A real integration loads the `using-code-distilling` bootstrap at session start. The bootstrap is what causes the porting skills to auto-trigger when porting intent is detected. Without it, the skills are dead weight.

**The acceptance test.** Open a clean session in the new harness inside any project, with a reference repo cloned somewhere on disk. Send exactly (substitute a real path):

> Let's port the X feature from /path/to/reference-repo.

A working integration auto-triggers `analyzing-reference` before any code is written. Paste the complete transcript in the PR.

## How to develop a skill change

1. Fork the repo and create a feature branch.
2. Edit the skill (`skills/<name>/SKILL.md` and any references).
3. **Test it.** Open a clean session, clone a small reference repo somewhere on disk, point the agent at its path, and walk through a port. Verify:
   - The right skills fire at the right moments.
   - The skill's Red Flags actually trip when you violate them.
   - The skill's checklist items get done.
4. Run the acceptance test on at least one supported harness.
5. Fill in the PR template completely.
6. Have a human review the complete diff.
7. Open the PR.

## Code of conduct

By participating, you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

By contributing, you agree your contributions will be licensed under the MIT License (see [`LICENSE`](LICENSE)).
