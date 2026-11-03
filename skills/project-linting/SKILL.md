---
name: project-linting
description: Does basic linting for the project which is not business-logic-specific.
metadata:
  purpose: "Maintenance for repositories."
  tags: maintenance, linting
  version: 1.0.3
---

# Project Linting

Do the following things in this repository (not in the submodules).

## Language

- Fix all typos in the code and documentation using the `fix-typos`-skill.
- Desired language in general: American English. Translate if necessary (except for business-logic-specific in translation-files).
- Use the write-documentation-skill.
- If dependencies from NPM are used then ensure that the used dependencies and dev-dependencies are pinned to a specific version. This rule does not apply to peer-dependencies.
- If dependencies from NuGet are used then ensure that the used dependencies are pinned to a specific version.
- If the project defines linting-rules ensure they are applied everywhere where applicable in the repository.
- If the project defines linting-commands or linting-scripts then run them and fix the valid findings.
- If the findings include false-positives then try to suppress or ignore them using the mechanism of the linting-tool which generated these false-positive findings.
- Format markdown-files using the `format-markdown-file`-skill.

## Coding

- Use the `mailmap-cleanup`-skill.
- If dependencies from PyPI are used then ensure that the required versions are pinned to a minimal version using ">=".
- If dependencies from NuGet are used then ensure that the used dependencies are pinned to a fix version.
