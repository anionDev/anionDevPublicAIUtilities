---
name: plan-fix-for-dotnet-nullcheck-warnings
description: Proposes fixes for warnings regarding null and null-references in .NET-projects per affected location. Read-only analysis, this skill does not change any file.
metadata:
  purpose: "Maintenance for repositories."
  tags: maintenance, dotnet
  version: 2.0.0
---

# Skill plan-fix-for-dotnet-nullcheck-warnings

## Goal

Analyzes nullable and NullReference warnings in C# projects systematically and proposes a technically correct fix for each affected location.

This skill does not change any file. The result is a list of proposals which the user can review and implement.

## When to use

- When warnings like `CS8600` to `CS8777` occur during the build and the user wants to know how to fix them.
- When a user explicitly wants a plan or proposals for fixing "NullReference warnings".
- When nullable annotations are inconsistent and lead to potential runtime `NullReferenceException`s.

## Core principle

First determine all warnings completely, then decide per warning on a technical basis which fix to propose:

1. Can `null` actually occur from a domain point of view?
2. If yes: propose to model nullability correctly (`?`) and to ensure null-safe usage.
3. If no: propose to tighten the API/types so that `null` is no longer possible (remove nullability, add guards), without introducing errors.

## Inputs

- Path to the solution or the project.
- Current compiler warnings (build output).
- Domain rules on whether a value is actually optional.

## Workflow

1. Collect nullable warnings

- Do not edit the csproj-file. If `<Nullable>enable</Nullable>` is not set, then pass the property to the build instead (`-p:Nullable=enable`).
- Run the build with warnings (e.g. `dotnet build --no-incremental`). Only the usual build-output (e.g. `bin` and `obj`) may be created by this.
- Extract the relevant warnings from the build output.
- Map the warning codes against the Microsoft documentation:
	https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/nullable-warnings

2. Locate the cause per warning

- Open the file and line.
- Check the data flow: where does a potential `null` come from, where is it dereferenced or assigned?
- Group warnings which have the same cause, so that one proposal can cover all of them.

3. Choose a technically correct fix strategy

- Strategy A: `null` is allowed from a domain point of view
	- Propose the correct nullable annotation (`string?`, `MyType?`).
	- Propose null checks (`if (x is null) ...`, `ArgumentNullException.ThrowIfNull(...)` where appropriate).
	- Name the call sites and return types to which the change must be propagated.

- Strategy B: `null` is not allowed from a domain point of view (preferred)
	- Propose to remove nullability from signatures when the value must never be `null`.
	- Propose early guards so that the contract is clear.
	- Propose how to guarantee initialization (constructor, required members, sensible defaults).

4. Report

Output one proposal per location or per group of warnings with the same cause:

- Location (file and line).
- Warning code and message.
- Cause of the warning.
- Chosen strategy and reason for this choice.
- Proposed code change (before and after).
- Change of program-behavior which the proposal would cause, or the statement that there is none.
- Open question, if the domain rules do not make clear whether a value is optional. In this case do not guess, but name the alternatives.

Sort the proposals so that locations with a risk of a runtime `NullReferenceException` come first.

## Decision rules

- Prefer tightening the model over "silencing the warning".
- Propose `!` (null-forgiving operator) only as a last resort and only with a clear invariant which is named in the proposal.
- Proposals must respect API contracts; be especially careful with public APIs and mark breaking changes explicitly.
- With multiple possible fixes: propose the one with the least domain ambiguity.

## Typical fix patterns

- Secure dereferencing:
	- Before: `value.Length`
	- After: null check or safe alternative (`value?.Length`, if technically correct)

- Tighten the parameter contract:
	- If `null` is not allowed: `string? input` -> `string input` + guard
	- If `null` is allowed: `string input` -> `string? input` + defensive usage

- Correct the return contract:
	- If the method can return `null`, mark the return type as nullable.
	- If not, adjust the implementation so that `null` is never returned.

## Anti-patterns

- Changing, creating or deleting source-files or project-files.
- Proposing `#nullable disable` across the board.
- Proposing to suppress warnings instead of fixing the cause.
- Blindly proposing `?` everywhere even though non-null holds from a domain point of view.

## Short checklist for the agent

- Warnings pulled from the build?
- Checked against the Microsoft reference?
- Decided per location on a technical basis: allow or remove nullability?
- Proposal available for each warning?
- Changes to program-behavior named in the report?
- No file changed?
