---
name: change-requirement-implement
description: Skill for implementing change requirements. Only applicable for repositories which uses openspec.
disable-model-invocation: true
metadata:
  version: 1.2.0
---

The skill must be invoked with a given requirement-name.

Do the following things:

1. Do `git fetch` for all available origins.
2. Checkout working branch which contains the requirement-name and pull the latest changes from that branch if there are any.
3. Ensure preconditions are set. This means the usual build-pipeline-command including build-step and run-testcases-step must be working with no errors.
4. Verify specification using `openspec validate <change-name> --strict` and `openspec status --change <change-name>` if the specification is implementable.
5. Implement changes using `/opsx:apply <change-name>`. This step is finished when the requirement is implemented and the usual build-pipeline-command including build-step and run-testcases-step is working with no errors.
6. Verify changes using `/opsx:verify <change-name>`.
7. Archive changes using `/opsx:archive <change-name>`.
8. Suggest a git commit-message for the change.
9. Ask the user if he wants the branch to be pushed to the remote repository.
10. If the user agrees, push the branch to the remote repository.
