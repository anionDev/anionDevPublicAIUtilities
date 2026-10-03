---
name: format-markdown-file
description: Formats a markdown file.
metadata:
  purpose: "Formats markdown files."
  tags: linting, formatting
  version: 1.0.0
---

# Format Markdown File

A markdown file is correctly formatted if it adheres to the following rules:

- One sentence per line. (Except in code blocks, tables, and lists.)
- No trailing whitespace.
- No more than one empty line in a row.
- One empty line before and after a code block, table, headline and list.
- Table-cells have the same length in each column (padded with whitespace to match the longest cell in the column, this also applies to help-lines like `| --- | --- |`).
- A multiline codeblock provides a language identifier after the opening backticks and uses "text" if the language is unknown or if no suitable language is available.
