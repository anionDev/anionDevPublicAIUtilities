---
name: validate-specs
description: Audit the current codebase against ALL main OpenSpec specs (openspec/specs/), not just one change. Use when the user asks to check whether the code currently satisfies the specs or says "validate-specs" or says "/opsx:audit", or wants a drift report between specs and implementation. Read-only.
metadata:
  version: 1.0.0
---

# Validate Specs

Check whether the current implementation satisfies every requirement and scenario in the
main specs under `openspec/specs/`. This complements `/opsx:verify`, which only checks a
single change against its own artifacts.

**This skill is read-only. Never edit code, specs, or tasks while auditing.**

## Input

- No argument: audit all capabilities in `openspec/specs/`.
- One or more capability names (e.g. `auth payments`): audit only those.
- Optional `--report`: also write the report to a file (see Output).

## Steps

### 1. Discover scope

1. List capabilities. Prefer the CLI, fall back to the filesystem:
   - `openspec list --specs` (use `--json` if supported)
   - otherwise: every `openspec/specs/<capability>/spec.md`
2. Read `openspec/config.yaml` and `openspec/project.md` / `AGENTS.md` if present, for
   project conventions (language, layout, test framework).
3. List active changes (`openspec list`, or folders in `openspec/changes/` except `archive/`).
   Note their names: code that implements an unarchived change may legitimately differ
   from the main specs. Do not count that as a failure; report it separately.

If there are no main specs, stop and tell the user.

### 2. Extract requirements

For each capability, parse every `### Requirement:` and its `#### Scenario:` blocks
(WHEN/THEN/AND). Keep the exact requirement title as the identifier in the report.

### 3. Check each requirement against the code

For every requirement:

1. Locate the implementing code (search by domain terms, identifiers, routes, CLI flags,
   config keys mentioned in the requirement). Record file paths with line numbers.
2. Evaluate each scenario individually: does the code produce the THEN outcome under the
   WHEN conditions? Trace the actual code path; do not infer from names alone.
3. Look for tests covering the scenario. Record them, or note that none exist.
4. Assign a status per scenario:
   - **PASS** – implemented as specified, with concrete evidence (file:line).
   - **PARTIAL** – implemented, but an edge case, condition, or AND-clause is missing or differs.
   - **FAIL** – not implemented, or behavior contradicts the spec.
   - **UNVERIFIED** – could not be determined statically (e.g. depends on runtime,
     external service, config not in repo). Say what would be needed to check it.

Rules:
- Never mark a scenario PASS without evidence. When in doubt, use UNVERIFIED.
- Never report a skipped or unchecked scenario as passing.
- If the spec itself is ambiguous or contradictory, say so instead of guessing.
- Do not run commands that change state. Running the existing test suite is allowed
  only if the user asked for it or it is clearly cheap and side-effect free; report
  test results as supporting evidence, not as a replacement for the scenario check.

For large repos with many capabilities, audit capabilities in parallel with subagents if
the agent supports it (one subagent per capability, each returning the per-scenario
table), then merge.

### 4. Optional: undocumented behavior

If time allows, note significant user-facing behavior found during the search that no
spec describes (new endpoints, commands, options). List these as SUGGESTION only.

## Output

Respond in the user's language. Print the table in the following structure:

```
## OpenSpec Audit – <date>

**Scope:** <n> capabilities, <n> requirements, <n> scenarios
**Result:** <n> PASS · <n> PARTIAL · <n> FAIL · <n> UNVERIFIED

### Findings
CRITICAL   – FAIL scenarios (spec says X, code does Y / nothing)
WARNING    – PARTIAL scenarios, scenarios without any test
SUGGESTION – UNVERIFIED items, ambiguous specs, undocumented behavior

### Per capability
#### <capability>
| Requirement | Scenario | Status | Evidence | Test |
| ----------- | -------- | ------ | -------- | ---- |

### Active changes affecting this audit
<change name> – <which requirements it touches, expected divergence>
```

Each CRITICAL/WARNING finding gets: requirement, scenario, what the spec says, what the
code does, file:line, and a one-line suggested next step (fix code via a new change, or
update the spec if the code is right).

## After the audit

Do not fix anything automatically. Offer next steps:
- Fix drift: start a change with `/opsx:new` for the FAIL/PARTIAL items.
- Spec is outdated: propose a change that updates the spec to match intended behavior.
- Re-run `/opsx:audit` after changes are archived.
