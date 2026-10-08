---
name: change-requirement-implement
description: Skill for implementing change requirements. Only applicable for repositories which uses openspec.
metadata:
  version: 1.0.0
---
The skill must be invoked with a given requirement-name.

Do the following things:

- Checkout working branch.
- Verify specification with `openspec validate <name> --strict` and `openspec status --change <name>` if the specification is implementable.
- Implement changes with `/opsx:apply <name>`. This step is finished when when the requirement is implemented and the usual build-pipeline-command (including build-step and run-testcases-step) is working with no errors.
- Verify changes with `/opsx:verify <name>`.
- Archive changes with `/opsx:archive <name>`.
- Suggest a git commit-message for the change.
