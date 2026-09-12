---
name: refine
description: Refine an idea together with the user and turn the result into a specification and into tickets which are ready to be implemented by an AI-agent.
disable-model-invocation: true
metadata:
  purpose: "Refinement of ideas into implementable specifications and tickets."
  tags: productivity, refinement, specification
  version: 1.0.0
---

# Skill refine

## Goal

Turn a rough idea of the user into a specification and into a set of tickets which are clear enough to be implemented by an AI-agent afterwards.

## Preconditions

- The skills `grill-with-docs`, `to-spec` and `to-tickets` must be available. If one of them is not available, then refuse the entire request.
- Do not change any source-code and do not implement anything. The only files which may be written are the documentation-files created by `grill-with-docs` (`CONTEXT.md`, ADRs), the specification and the ticket-files.

## Configuration for the sub-skills

`to-spec` and `to-tickets` expect that an issue-tracker and a triage-label-vocabulary were provided to them. This skill provides them. Do not ask the user to run `/setup-matt-pocock-skills`.

- The specification is stored as openspec-change if the repository contains an `openspec`-folder, otherwise in `<repository>/.scratch/<feature-slug>/spec.md`.
- Tickets are stored as one file per ticket in `<repository>/.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency-order (blocking tickets first). Never write a single combined ticket-file.
- The triage-state of a ticket is a `Status:`-line near the top of the ticket-file.
- The triage-label-vocabulary consists of exactly these labels: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`.

## Process

1. Use the `grill-with-docs`-skill to refine the idea given by the user. Continue until the user confirms that a shared understanding is reached.
2. Use the `to-spec`-skill to write the specification, but only to produce the specification-text. `to-spec` must not publish the specification itself, because step 3 decides where the specification is stored.
3. Write the specification down:
   - If the repository contains an `openspec`-folder: use the `openspec-propose`-skill and pass the specification from step 2 to it as input. Its own clarification-interview is skipped, because step 1 already covered it. Its rule to stop after the planning-artifacts does not end this skill, because step 4 belongs to the refinement and does not change any source-code.
   - Otherwise: write the specification to `<repository>/.scratch/<feature-slug>/spec.md`.
4. Use the `to-tickets`-skill to create the tickets for this specification at the location defined above. Derive the tickets from the specification of step 3 (including the tasks which openspec generated, if openspec was used) instead of breaking the idea down again from scratch. In every ticket add a reference to the specification the ticket belongs to: the name of the openspec-change, or the path of the specification-file.
5. Print a summary which contains the location of the specification, the list of the created tickets and the points which are still unresolved.
