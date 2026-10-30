---
name: search-for-findings
description: Searches for findings in the current repository.
disable-model-invocation: true
metadata:
  tags: security, linting
  version: 1.0.0
---

Search for any kind of findings in the current repository.
If there is one central finding list for the current repository, add all discovered findings to that list.

A finding is:

- Any security vulnerability discovered in the repository.
- Any linting issue found in the codebase.
- Any other issue/bug/performance-bottleneck that could potentially affect the quality, security, or maintainability of the project.
- Any typos found in public functions/variables/APIs. (For private functions/variables, fix them immediately instead of reporting them as findings. Use the `fix-typos`-skill.)

Except of the typos in private functions/variables, all other findings should only be reported, not fixed.
