---
name: change-requirement-create
description: Skill for creating change requirements. Only applicable for repositories which uses openspec.
disable-model-invocation: true
metadata:
  version: 1.1.0
---
The skill must be invoked with a given requirement-name.

Do the following things:

- Do `git fetch` for all available origins.
- Create and checkout working branch matching the requirement-name from the latest main-branch-commit.
- Ensure preconditions are set. This means the usual build-pipeline-command (including build-step and run-testcases-step) must be working with no errors.
- Create change as specification using `/opsx:new <name>`
- Ask the user to write down the full specification. Suggest the user to Consider using the skills `grill-with-docs` and `to-spec` and `to-tickets` for that. The tickets must be saved in the openspec-change.
- Check specification using `openspec validate <name> --strict`.
- Let the user review the results and optionally do changes.
- If the specification is valid and reviewed, suggest a git commit-message for the change.
- Ask the user if he wants the branch to be pushed to the remote repository.
- If the user agrees, push the branch to the remote repository.
