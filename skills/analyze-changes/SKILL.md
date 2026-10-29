---
name: analyze-changes
description: Analyze the changes on the current branch.
metadata:
  purpose: "Analysis for repositories."
  tags: analysis, git
  version: 1.0.1
---

# Analyze Changes

Summarize the changes in this repository in short.

# Uncommitted Changes

If there are currently uncommitted changes in the repository:
Describe the uncommitted changes.

## Commit Suggestion

In the end, as last output-line write a suggestion for a good one-line git-commit-command for all changes which are still uncommitted including a good commit-message for it.
The commit-subject should be a short summary of the changes in this branch.
Optionally you can add a commit-body to the suggestion which describes the changes in more detail.
Commit-message and commit-body should be written in American English.

# No uncommitted changes

If there are no uncommitted changes:
Describe the changes on the current branch compared to the main branch.
