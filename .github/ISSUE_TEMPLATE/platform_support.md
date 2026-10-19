---
name: Platform / harness support
about: Request or contribute support for a new agent harness (IDE, CLI, agent runner)
title: "[platform] "
labels: platform-support
assignees: ''
---

## Which harness?

<!-- Name, link to docs. -->

## How does it load skills?

<!-- Does it support plugin manifests? Slash commands or skill mentions?
     Can a skill be limited to explicit invocation?
     Paste links to its plugin docs. -->

## What needs to change?

<!-- Manifest file? Per-skill invocation policy? Be specific. -->

## Acceptance test plan

<!-- A real integration never loads the skills on its own, and reaches
     `distillation-spec` when the user explicitly invokes
     `using-code-distilling`. Describe how you'd verify both in the new
     harness. -->

## Are you willing to send a PR?

- [ ] Yes, I can implement and provide a session transcript proving the
      explicit invocation works and nothing loads without it.
- [ ] No, but I can help test.
- [ ] No.
