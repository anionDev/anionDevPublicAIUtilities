---
name: fix-finding
description: Fixes a finding.
metadata:
  purpose: "Provide steps to work on findings."
  tags: development
  version: 1.0.0
disable-model-invocation: true
---

This skill must be called with a given finding/issue.
If any detail is missing so that it is not clear how the task is supposed to be implemented then refuse the implementation due to missing information.

If all required information is available then do the following things:

- Implement the fix in a clean way.
- If available, update any relevant documentation due to the changed behavior.
- If possible in a sensible way, write testcases for the fix.
- If the finding is listed in the finding-list in this repository then mark it as resolved.
- Do not change any behavior unrelated to the finding.
- Commit the changes in the end.

If the user wants multiple findings to be fixed then do the list above sequentially for each finding:
Every finding gets exactly one commit which includes the fix and maybe documentation and/or testcases and nothing else.
