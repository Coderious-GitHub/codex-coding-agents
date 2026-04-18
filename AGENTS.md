# AGENTS.md

## Working model

This repository uses a staged delivery workflow for non-trivial work.

The standard loop is:

1. Architect creates the global plan and the current pass brief
2. Implementer executes the current pass only
3. Implementer stops and returns structured feedback
4. Architect updates the plan and writes the next pass
5. Repeat until all passes are complete
6. Reviewer performs a final audit

For trivial one-file fixes, the plan may be short.
For architectural, multi-file, or system-level work, always plan before implementation.

---

## Role boundaries

### Architect
Owns:
- global plan
- pass sequencing
- architecture decisions
- scope boundaries
- tradeoff decisions
- deciding whether findings require a change of plan

Does not:
- silently drift into implementation unless explicitly asked
- approve speculative redesigns without evidence
- write broad future-pass instructions inside the current pass

### Implementer
Owns:
- executing the current approved pass
- making the smallest viable code changes for that pass
- validating the current pass
- returning structured feedback to the architect

Does not:
- implement future passes
- silently redesign the system
- widen scope because it "seems cleaner"
- decide the next pass
- perform unrelated refactors

### Reviewer
Owns:
- final or milestone-based audit
- correctness
- regressions
- missing validation
- maintainability risks

Does not:
- rewrite the plan
- perform style-only review unless it hides a real issue

---

## General working rules

- Prefer minimal, localized diffs over speculative cleanup.
- Preserve existing behavior and public contracts unless the task explicitly allows change.
- Do not silently change APIs, payloads, serialization, save formats, scene wiring, or configuration formats.
- Follow existing patterns unless there is a clear reason not to.
- Prefer designs and refactors that align with SOLID principles where they fit the existing codebase and scope.
- Apply SOLID pragmatically: improve cohesion, reduce unnecessary coupling, and avoid forcing new abstractions when a smaller local change is more appropriate.
- Inspect real code paths before proposing abstractions.
- Do not mix architecture, gameplay logic, UI, persistence, and tests in one pass unless necessary.
- Each pass should leave the repo in a working state whenever reasonably possible.
- Never declare work done without stating what was actually validated.

---

## Standard planning template

The architect should structure plans using this shape:

- Objective
- Scope
- Out of scope
- Architecture requirements
- Suggested structure
- Functional flow
- Requirements by subtask/module, including error handling
- Dependencies
- Validation
- Summary of what has been done
- Decisions needed before next pass

For larger work, also include:
- Risks / unknowns
- Definition of done

---

## Architect output contract

When the architect writes a pass, it must include:

### Pass header
- Pass number
- Pass name
- Objective of this pass

### In scope
Exactly what this pass is allowed to change.

### Out of scope
Exactly what this pass must not change.

### Architecture requirements
Design constraints and invariants that must be respected.

### Suggested structure
Expected files, modules, services, interfaces, or classes involved in this pass.

### Functional flow
Describe the relevant runtime or data flow for this pass only.

### Requirements by subtask/module
For each affected piece:
- responsibility
- inputs
- outputs
- expected behavior
- error handling
- edge cases

### Dependencies
Relevant systems, data, services, or modules this pass depends on.

### Validation
How this pass should be checked.

### Expected implementer feedback
What the implementer must report back at the end of the pass.

### Stop condition
The implementer must stop after this pass and wait for new architect instructions.

---

## Implementer output contract

At the end of every pass, the implementer must return a structured handoff using this shape:

- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

Important:
- Findings are facts discovered during implementation.
- Proposed adjustments are not decisions.
- The implementer may suggest options, but the architect decides the next pass.

---

## Validation rules

Validation must be proportionate to the pass.

Possible validation includes:
- compile/build success
- relevant automated tests
- targeted manual verification
- smoke checks for critical flows
- logging/error-path verification
- proof that existing behavior remains intact where required

If validation was not run, say why clearly.

---

## Definition of done

A task is only done when:
- all planned passes are complete or explicitly closed
- requested behavior is implemented
- scope boundaries were respected
- validation was performed or the gap is clearly stated
- known risks are surfaced
- reviewer audit has been completed for non-trivial work

---

## Project-specific notes

This repository is a template for staged multi-agent Codex workflows.

### High-risk areas
- pass boundaries being skipped
- architect instructions drifting into implementation
- implementer widening scope beyond the current pass
- plan files in `docs/plans/` falling out of sync with actual progress
- reviewer audits being skipped or replaced with self-approval
- findings and proposed adjustments being mixed together in handoffs
- changing templates or agent prompts in ways that break role separation

### Common commands
- Validate repo structure:
  - `Get-ChildItem`
  - `rg --files`
- Read workflow rules:
  - `Get-Content AGENTS.md`
  - `Get-Content docs\templates\task.md`
  - `Get-Content docs\templates\architect-pass-template.md`
  - `Get-Content docs\templates\implementer-feedback-template.md`
- Inspect agent definitions:
  - `Get-Content .codex\config.toml`
  - `Get-Content .codex\agents\architect.toml`
  - `Get-Content .codex\agents\implementer.toml`
  - `Get-Content .codex\agents\reviewer.toml`

### Repo constraints
- keep the architect, implementer, and reviewer responsibilities clearly separated
- do not skip directly from planning to full implementation on non-trivial work
- keep `docs/plans/<plan-name>.md` current throughout execution
- the architect may write only the current pass in full detail
- the implementer must stop after the current pass and return structured feedback
- the reviewer is read-only and audits only after implementation passes are complete or at an explicit milestone
- preserve the templates in `docs/templates/` as the canonical workflow contracts unless intentionally updating the template itself
- prefer minimal edits that improve the template without changing its core staged model
