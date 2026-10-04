---
name: project-linting
description: Does basic linting for the project which is not business-logic-specific.
metadata:
  purpose: "Maintenance for repositories."
  tags: maintenance, linting
  version: 1.0.2
---

Do the following things in this repository (not in the submodules):

- Fix all typos in the code and documentation. Ignore files which are generated or git-ignored or have binary-content.
- Desired language in general: American English. Translate if necessary. (Except for business-logic-specific content which is not in the scope of this skill.)
- Use the write-documentation-skill.
- If dependencies from NPM are used then ensure that the used dependencies and dev-dependencies are pinned to a specific version. This rule does not apply to peer-dependencies.
- If dependencies from NuGet are used then ensure that the used dependencies are pinned to a specific version.
- If dependencies from PyPI are used then ensure that the used dependencies are pinned to a minimal version using ">=".

If the project defines linting-rules ensure they are applied everywhere where applicable in the repository.
If the project defines linting-commands or linting-scripts then run them and fix the findings.
Format mark-down-files using the `format-markdown-file`-skill.
