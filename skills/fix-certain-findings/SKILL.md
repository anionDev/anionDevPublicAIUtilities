---
name: fix-certain-findings
description: Skill to fix certain findings in a source-code-repository.
disable-model-invocation: true
metadata: 
  version: 1.0.0
---

# Fix Certain Findings

If there is no one clear central finding-list: Do nothing.

If there is a clear central finding-list:

- Ensure there are no uncommitted changes by committing them on the current or on another branch or by stashing them.
- For each finding in the finding list do the following: If the finding can be automatically fixed without doubt and without requiring further input/information and without break anything and without changing public APIs do the following (otherwise ignore the finding and go to the next one):
  - Apply the fix.
  - If possible without introducing new issues, test the fix to ensure it works as intended. Do not write tests which require new libraries or access outside of the repository.
  - Mark the finding as fixed in the finding list.
  - Commit the changes related to the fixed finding.
