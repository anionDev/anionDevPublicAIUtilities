---
name: fix-small-bug
description: Do a small fix in a source-code-repository.
metadata:
  purpose: "Maintenance for repositories."
  tags: maintenance, ai-self-test
  version: 1.0.2
---

Implement one small fix in the repository if you find one.
Your change must fix a concrete clear bug if you found one and fulfill all requirements listed below.
(If not possible: do nothing instead of trying to fix that.)
Constraint for the change/fix:

- It must be very clear that the issue you fixed is really a bug.
- The fix must not break anything.
- The fix is not allowed to cause much effort or refactoring.

A typo is treated as a small bug.
If it is possible without much effort, test the fix to ensure it works as intended.
Do not write tests which require new libraries or access outside of the repository.
