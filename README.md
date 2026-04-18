# Codex Staged Coding Agent Template

This repository is a template for running Codex as a small staged delivery system instead of a single all-at-once coding agent.

It splits work across three roles:

- `architect`: plans the work, defines scope, and writes one implementation pass at a time
- `implementer`: executes only the current pass, validates it, and reports structured feedback
- `reviewer`: performs the final audit for correctness, regressions, missing validation, and maintainability risks

The core idea is simple: the architect and implementer work in a loop until the planned passes are complete, then the reviewer audits the result.

## How The Workflow Works

For non-trivial work, the loop is:

1. The architect inspects the codebase and writes the global plan.
2. The architect writes only the current pass in full detail.
3. The implementer reads that pass and executes only that pass.
4. The implementer stops and returns structured end-of-pass feedback.
5. The architect reviews the findings, updates the plan, and writes the next pass.
6. Repeat until all passes are complete.
7. The reviewer performs a read-only final audit.

This is defined by the repository guidance in `AGENTS.md` and by the per-agent instructions in `.codex/agents/`.

## Why This Template Exists

This template is meant to make Codex behave more like a disciplined engineering team:

- planning stays separate from implementation
- implementation stays narrow and reviewable
- findings discovered during coding feed back into the plan
- final review is independent from both planning and implementation

That keeps scope tighter and reduces the usual failure mode where an agent plans, implements, redesigns, and self-approves in one blurry step.

## Role Responsibilities

### Architect

The architect owns:

- the global plan
- pass sequencing
- architecture decisions
- scope boundaries
- tradeoff decisions
- deciding whether implementer findings should change the roadmap

The architect does not silently drift into coding and does not write multiple future passes as if they were already approved.

In this template, the architect is defined in [.codex/agents/architect.toml](.codex/agents/architect.toml) and is configured as a high-reasoning, read-only planning agent.

### Implementer

The implementer owns:

- executing the current approved pass
- making the smallest viable code changes for that pass
- validating the pass
- returning structured feedback to the architect

The implementer does not decide future passes, widen scope, or redesign the system because another approach looks nicer mid-stream.

The implementer prompt lives in [.codex/agents/implementer.toml](.codex/agents/implementer.toml).

### Reviewer

The reviewer owns:

- final or milestone audit
- correctness checks
- regression detection
- validation/test gaps
- maintainability risks

The reviewer is read-only and is not supposed to rewrite the plan or take over implementation.

The reviewer prompt lives in [.codex/agents/reviewer.toml](.codex/agents/reviewer.toml).

## Repository Structure

The main files in this template are:

- [AGENTS.md](AGENTS.md): repository-wide workflow rules, role boundaries, planning contracts, validation rules, and definition of done
- [docs/templates/task.md](docs/templates/task.md): starting prompt template for running a staged task
- [docs/templates/architect-pass-template.md](docs/templates/architect-pass-template.md): format for each architect-written pass
- [docs/templates/implementer-feedback-template.md](docs/templates/implementer-feedback-template.md): required handoff format after each pass
- [docs/plans/issue-tracker.md](docs/plans/issue-tracker.md): an example full plan showing the workflow across multiple passes
- [.codex/config.toml](.codex/config.toml): Codex agent system settings, currently limiting the setup to `max_threads = 4` and `max_depth = 1`

## Planning Model

The architect plans in passes, not as one giant implementation brief.

For larger tasks, the plan should include:

- objective
- scope
- out of scope
- architecture requirements
- suggested structure
- functional flow
- requirements by subtask or module
- dependencies
- validation
- summary of what has been done
- decisions needed before next pass
- risks / unknowns
- definition of done

Then the architect writes only the current pass in full detail and explicitly tells the implementer to stop after it.

## Implementer Handoff Model

At the end of every pass, the implementer must report back with:

- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

That structure is important because it separates facts from proposals. The implementer can suggest changes, but the architect decides whether the plan changes.

## Final Review Model

Once all planned passes are complete, the reviewer audits the result and reports:

- confirmed issues
- likely risks
- missing validation
- optional improvements
- final assessment

This gives the workflow a real end-state check instead of assuming implementation equals completion.

## Typical Usage

The intended flow for a task looks like this:

1. Start from [docs/templates/task.md](docs/templates/task.md).
2. Describe the feature, fix, or system you want built.
3. Have the architect inspect the code and create or update a file under `docs/plans/`.
4. Let the architect write Pass 1 only.
5. Let the implementer execute Pass 1 only.
6. Feed the implementer handoff back to the architect.
7. Continue architect -> implementer until all passes are done.
8. Run the reviewer at the end.

The included [docs/plans/issue-tracker.md](docs/plans/issue-tracker.md) is a concrete example of what a completed staged workflow looks like in practice.

## What Counts As Done

According to `AGENTS.md`, work is only done when:

- all planned passes are complete or explicitly closed
- the requested behavior is implemented
- scope boundaries were respected
- validation was performed, or the gap is stated clearly
- known risks are surfaced
- the reviewer has completed the final audit for non-trivial work

## Notes

This template is intentionally conservative. It favors minimal diffs, explicit scope control, and reviewable passes over speed-by-chaos. For one-file or otherwise trivial work, the plan can be short. For multi-file, architectural, or risky work, the staged loop is the point.
