---
name: change-requirement-implement
description: Skill for implementing change requirements. Only applicable for repositories which uses openspec.
disable-model-invocation: true
metadata:
  version: 1.1.0
---
The skill must be invoked with a given requirement-name.

Do the following things:

- Do `git fetch` for all available origins.
- Checkout working branch which contains the requirement-name.
- Verify specification using `openspec validate <name> --strict` and `openspec status --change <name>` if the specification is implementable.
- Implement changes using `/opsx:apply <name>`. This step is finished when when the requirement is implemented and the usual build-pipeline-command (including build-step and run-testcases-step) is working with no errors.
- Verify changes using `/opsx:verify <name>`.
- Archive changes using `/opsx:archive <name>`.
- Suggest a git commit-message for the change.
- Ask the user if he wants the branch to be pushed to the remote repository.
- If the user agrees, push the branch to the remote repository.
