Read AGENTS.md first.

Use the staged multi-agent workflow only if this task is non-trivial or if the initial prompt explicitly asks for it.

If the work is trivial, such as a small typo fix or a narrowly scoped one-file change, do not spawn architect, implementer, and reviewer agents by default.

Project/task:
<Describe the module, feature, system, or problem>

Objective:
<What needs to be achieved>

Functionalities / requirements:
- <requirement 1>
- <requirement 2>
- <requirement 3>

Constraints:
- keep scope tight
- prefer minimal diffs
- do not silently refactor unrelated code
- preserve existing behavior unless explicitly changed
- keep the plan updated in docs/plans/<plan-name>.md

Workflow:
1. Spawn the architect agent.
2. Have the architect inspect the relevant code and create or update docs/plans/<plan-name>.md.
3. The architect must:
   - write the global plan
   - define passes
   - write Pass 1 only in full detail
   - include expected implementer feedback
   - explicitly instruct the implementer to stop after Pass 1
4. Spawn the implementer agent.
5. Have the implementer:
   - read docs/plans/<plan-name>.md
   - execute Pass 1 only
   - stop after Pass 1
   - return structured end-of-pass feedback
6. Feed the implementer feedback back to the architect.
7. Have the architect:
   - update docs/plans/<plan-name>.md based on the feedback
   - preserve the overall plan unless the findings justify a change
   - write the next pass only
8. Repeat architect -> implementer until all passes are complete.
9. Spawn the reviewer agent at the end, or at an explicit milestone if the task calls for a milestone review.
10. Have the reviewer audit the current result against the plan.

Architect rules:
- owns the plan
- owns sequencing
- decides whether findings change the roadmap
- writes one pass at a time

Implementer rules:
- executes one pass only
- does not decide future passes
- reports findings, blockers, and proposed adjustments
- stops after each pass

Reviewer rules:
- read-only audit
- focus on correctness, regressions, missing validation, and maintainability risks

Output expectations:
- keep docs/plans/<plan-name>.md current
- show pass-by-pass progress
- do not skip directly from planning to full implementation
- do not continue past a pass boundary without architect instructions
