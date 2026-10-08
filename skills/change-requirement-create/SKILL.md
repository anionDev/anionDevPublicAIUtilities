---
name: change-requirement-create
description: Skill for creating change requirements. Only applicable for repositories which uses openspec.
metadata:
  version: 1.0.0
---
The skill must be invoked with a given requirement-name.

Do the following things:

- Create and checkout working branch.
- Ensure preconditions are set. This means the usual build-pipeline-command (including build-step and run-testcases-step) must be working with no errors.
- Create change as specification with `/opsx:new <name>`
- Ask the user to write down the full specification. Suggest the user to Consider using the skills `grill-with-docs` and `to-spec` and `to-tickets` for that. The tickets must be saved in the openspec-change.
- Check specification with `openspec validate <name> --strict`.
