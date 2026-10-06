---
name: format-markdown-file
description: Formats a markdown file.
metadata:
  purpose: "Formats markdown files."
  tags: linting, formatting
  version: 1.0.1
---

# Format Markdown File

While formatting a markdown file, the content and the meaning of the file must never be changed semantically.
Only whitespace, line breaks and table-cell padding may be adjusted.

A markdown file is correctly formatted if it adheres to the following rules:

- One sentence per line. (Except in code blocks, tables, and lists.) This applies in both directions: if two sentences are on one line, a line break is added after the first sentence; if one sentence is spread over multiple lines, the line breaks within the sentence are removed. This only applies where it is possible. For example if multiple sentences must be in one table-cell and therefore on one line, this is acceptable.
- No trailing whitespace.
- No more than one empty line in a row.
- One empty line before and after a code block, table, headline and list.
- Table-cells have the same length in each column (padded with whitespace to match the longest cell in the column, this also applies to help-lines like `| --- | --- |`).
- A multiline codeblock provides a language identifier after the opening backticks and uses "text" if the language is unknown or if no suitable language is available.
- Do not change texts which come clearly from external, for example generated files or texts with copyright notices and common license-texts.
- Never change a license-file, unless you are explicitly instructed to do so.
