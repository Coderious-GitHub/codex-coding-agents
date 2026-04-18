# Issue Tracker Staged Workflow Plan

## Objective
Build a minimal but complete issue tracker in the empty repo with a React frontend, a FastAPI backend, and SQLite persistence. The work must follow the staged architect -> implementer -> architect -> implementer -> reviewer workflow and stay within the requested scope.

## Scope
- View a list of issues
- Filter issues by status
- Create a new issue
- Edit an existing issue
- Mark an issue as done
- Backend validation for invalid payloads
- Frontend loading, empty, success, and error states
- SQLite persistence
- Basic automated backend tests
- Small but meaningful frontend coverage

## Out of scope
- Authentication
- Deployment work
- Websockets
- File uploads
- Background jobs
- Pagination unless the architecture forces it
- Broad unrelated refactors
- Any silent change to the issue contract

## Architecture requirements
- Keep the repo split into separate top-level `backend/` and `frontend/` apps
- Keep the issue contract stable across all passes
- Use the same field names and enum values on both sides
- Keep request and response shapes simple and explicit
- Prefer refresh-after-success behavior on the frontend instead of optimistic updates
- Keep the codebase small, readable, and easy to extend pass by pass
- Do not introduce abstractions that later passes must unwind

## Suggested structure
- Backend: `backend/app/main.py`, `backend/app/schemas.py`, `backend/app/api/`, `backend/app/core/`, `backend/tests/`
- Frontend: `frontend/src/main.tsx`, `frontend/src/app/`, `frontend/src/pages/`, `frontend/src/api/`, `frontend/src/types/`, `frontend/src/components/`
- Plan file: `docs/plans/issue-tracker.md`

## Functional flow
- Pass 1 creates the scaffold and locks the canonical contract
- Pass 2 implements backend CRUD, validation, timestamps, and SQLite persistence
- Pass 3 implements the frontend read flow and issue filtering
- Pass 4 implements create/edit/done interactions and mutation feedback
- Pass 5 tightens tests, cleanup, and final validation before reviewer audit

## Requirements by subtask/module

### Backend contract
- Responsibility: define the canonical issue schema and payload shapes
- Inputs: create/update request payloads and filter parameters
- Outputs: issue JSON objects and validation errors
- Expected behavior: the backend and frontend agree on `id`, `title`, `description`, `status`, `priority`, `created_at`, and `updated_at`
- Error handling: invalid status, invalid priority, malformed JSON, and missing required fields must be rejected later with clear 4xx responses
- Edge cases: whitespace-only title, empty description, and timestamp serialization

### Frontend contract
- Responsibility: mirror the backend issue shape and drive UI state
- Inputs: issue lists, single issue responses, mutation responses, and error payloads
- Outputs: list rows, forms, and state indicators
- Expected behavior: loading, empty, success, and error states must be visible and deterministic
- Error handling: failed fetches and validation errors must be shown to the user
- Edge cases: no issues, one issue, long titles, and status-specific filters

### Persistence and API
- Responsibility: store issues locally and expose the issue API
- Inputs: validated create/update payloads and issue lookups
- Outputs: persisted rows and stable IDs/timestamps
- Expected behavior: implement `GET /issues`, `GET /issues/{id}`, `POST /issues`, and `PUT /issues/{id}`
- Error handling: missing records and invalid payloads must return explicit errors
- Edge cases: empty database, first-run schema creation, and updates that do not change every field

## Dependencies
- Empty `backend/` and `frontend/` directories
- Default tooling assumptions unless the repo already suggests another convention
- FastAPI and SQLite on the backend
- Vite + React + TypeScript on the frontend
- Pass 2 must begin by bootstrapping the backend validation environment if `pytest` and `httpx` are missing
- Pass 3 must begin by bootstrapping the frontend validation environment if `node_modules` or the required scripts are missing; `npm install` and the validation commands used to prove the read flow are expected setup work in that pass, and any command that needs runtime writes may require escalation in this workspace

## Validation
- Pass 1: scaffold imports, startup, build, or type-check succeed
- Pass 2: backend tests cover happy path and invalid payloads
- Pass 3: frontend read flow shows loading, empty, success, and error states
- Pass 4: frontend mutation flow covers create, edit, and mark-done behavior
- Pass 5: full relevant test run and reviewer audit against the plan

## Summary of what has been done so far
- Repo guidance in `AGENTS.md` and the templates in `docs/templates/` have been read
- The repo state has been inspected
- `backend/` and `frontend/` are empty
- No implementation work has started yet
- This plan file has been created to guide the staged workflow
- Pass 1 scaffold has now been implemented with placeholder backend routes, mirrored issue contracts, and frontend page/component shells
- Scaffold metadata has been added for a FastAPI backend and a Vite + React + TypeScript frontend
- Backend syntax compilation without bytecode output succeeded, and the FastAPI app imported with the planned issue routes registered
- Backend test execution could not run because `pytest` is not installed in the current Python environment, and `fastapi.testclient` currently lacks its `httpx` dependency
- Frontend type-check validation was initially blocked because frontend dependencies had not been installed yet; that blocker has since been cleared and the frontend toolchain is now available locally
- Pass 1 assumptions matched the implementation: FastAPI + SQLite on the backend, Vite + React + TypeScript on the frontend, and no separate shared package
- Pass 2 is completed; it replaced the backend `501` stubs with a SQLite-backed issue API covering list, detail, create, update, status filtering, timestamps, and explicit `404` behavior for missing records
- Backend payload validation now rejects invalid enums, empty or whitespace-only titles, null update fields, and empty update payloads while preserving the Pass 1 contract
- Backend validation bootstrap was required and completed by installing the backend dev dependencies, after which the Pass 2 API test suite passed
- Backend tests pass when run outside the sandbox; sandboxed execution still blocks runtime temp and SQLite file writes
- Frontend dependencies were installed successfully after the earlier Pass 3 blocker, so the read-flow pass can resume without a bootstrap step unless the local toolchain becomes unusable again
- Pass 3 is completed; the frontend issue list now fetches `GET /issues`, forwards the selected status as a server-side query parameter, and renders distinct loading, empty, success, and error states
- Frontend validation now includes a small Vitest + Testing Library read-flow suite covering success, empty, error, and filtered-list behavior
- Frontend test validation required adding frontend test dependencies and running Vitest outside the sandbox because the workspace blocked the test runner spawn path under sandboxed execution
- Pass 3 confirmed that no backend contract or route change was needed, and the current frontend scaffold is sufficient for mutation work in Pass 4
- Pass 4 is completed; the frontend now supports create, edit, and mark-done flows in the existing page shell, reuses the right-side `IssueFormShell` for edit mode, surfaces frontend and backend validation errors, and refreshes the list after successful mutations
- Frontend mutation coverage now exercises create, edit, mark-done, frontend validation, and backend validation-error display without changing the backend contract
- Pass 4 validation succeeded with `npm run typecheck`, `npm run build`, and `npm run test`; frontend tests no longer needed escalation in the current workspace during this pass
- Pass 5 is completed; backend coverage now also proves SQLite persistence across app restarts, malformed JSON rejection, and extra-field validation while the contract and route surface remain unchanged
- Final validation succeeded with `python -m pytest tests/test_issues_api.py`, `npm run test`, `npm run typecheck`, and `npm run build` in the current workspace without extra bootstrap or escalation
- Temporary runtime artifacts are now treated as cleanup-only outputs rather than source files, and the repo metadata no longer describes the backend as scaffold-only

## Decisions needed before next pass
- No product decision is blocking the roadmap
- Pass 2 explicitly included backend dependency/bootstrap setup as the first validation step, so the implementer did not need to decide whether to install `pytest` and `httpx`
- Architect decision for Pass 3: frontend dependency installation and the `npm run typecheck` / `npm run build` validation commands are expected setup work at the start of the pass if the current workspace does not already have a usable frontend toolchain
- Architect decision for Pass 4: reuse the existing right-side `IssueFormShell` area for both create and edit mode, with edit mode selected from the issue list and the form content swapping in place rather than introducing a separate edit screen or drawer
- Pass 4 stayed within the planned mutation scope and did not require backend contract changes or a second edit surface
- Architect decision for Pass 5: focus primarily on backend validation and persistence edge cases plus cleanup and reviewer prep; add only a minimal frontend regression test if a real gap is discovered while tightening the final pass
- No further implementation pass is planned; the next step is reviewer audit against the completed plan
- Pass 5 stayed within the planned cleanup/test scope and did not require an extra frontend regression test

## Risks / unknowns
- The repo starts empty, so tooling, scripts, and test setup must be introduced cleanly
- The scaffold pass exposed missing validation dependencies in the current environment
- Later passes depend on keeping the contract stable from the start

## Definition of done
- All planned passes are complete or explicitly closed
- The requested issue tracker behavior is implemented
- Tests are added and run as planned
- Scope boundaries are respected
- Reviewer audit is completed

# Pass 1 - Scaffold Backend/Frontend Structure and Shared Contract

**Status:** Completed

## Objective
Create runnable backend and frontend scaffolds and lock the canonical issue contract so later passes can implement persistence and UI behavior without changing the core shape.

## In scope
- Create the minimal project structure for `backend/` and `frontend/`
- Add app entrypoints and the smallest useful build or package metadata
- Define mirrored issue enums, types, and schemas on both sides
- Add placeholder backend route modules and frontend page/component shells
- Add only scaffold-level validation such as import, startup, build, or type-check

## Out of scope
- SQLite persistence
- Real CRUD logic
- Live data fetching or mutation wiring
- Backend validation beyond contract definitions
- Frontend data loading or mutation behavior
- Business-logic tests

## Architecture requirements
- Keep the issue field names fixed: `id`, `title`, `description`, `status`, `priority`, `created_at`, `updated_at`
- Keep the status values fixed: `todo`, `in_progress`, `done`
- Keep the priority values fixed: `low`, `medium`, `high`
- Keep the scaffold flat and easy to extend
- Avoid behavior that later passes would need to unwind
- Do not add a separate shared package for the contract

## Suggested structure
- Backend: `backend/app/main.py`, `backend/app/schemas.py`, `backend/app/api/`, `backend/app/core/`, `backend/tests/`
- Frontend: `frontend/src/main.tsx`, `frontend/src/app/`, `frontend/src/pages/`, `frontend/src/api/`, `frontend/src/types/`, `frontend/src/components/`
- Keep any repository-level metadata limited to what is needed to make the scaffold installable and testable

## Functional flow
- The backend starts with a minimal FastAPI entrypoint and importable schema definitions
- The frontend renders a basic issue tracker page shell with typed API boundaries
- Both sides agree on the issue shape before persistence or network behavior exists

## Requirements by subtask/module

### Repository scaffold
- Responsibility: create the app roots and baseline metadata
- Inputs: empty `backend/` and `frontend/` directories
- Outputs: runnable project skeletons with minimal scripts
- Expected behavior: a fresh clone can identify where backend and frontend code live without ambiguity
- Error handling: if tooling choices need to change, keep the contract stable and adjust only scaffold files
- Edge cases: no pre-existing package manager files, no test scripts, and no lint conventions

### Backend scaffold
- Responsibility: define the FastAPI app shell and the canonical backend schema modules
- Inputs: issue field definitions and route names
- Outputs: importable app entrypoint, schema and enum definitions, and placeholder router modules
- Expected behavior: the app can start once dependencies are installed, even though data access is not implemented yet
- Error handling: invalid or missing implementation details must be deferred to Pass 2 rather than improvised now
- Edge cases: empty database file, missing issue IDs, and absent persistence are intentionally not handled in this pass

### Frontend scaffold
- Responsibility: define the React app shell and the mirrored frontend types
- Inputs: the canonical issue contract and UI state placeholders
- Outputs: page shell, component stubs, and typed API client boundaries
- Expected behavior: the app renders a basic issue tracker page without assuming live data
- Error handling: loading, empty, and error states should be represented in the component structure even before live wiring
- Edge cases: no issues available and failing fetches are reserved for later passes

### Shared contract
- Responsibility: lock the shape of issues before business logic is added
- Inputs: the canonical field list and enum values
- Outputs: mirrored backend schemas and frontend TypeScript types
- Expected behavior: both sides use the same names and values without hidden translation layers
- Error handling: invalid shapes are left for later validation work, but the contract should make them easy to reject
- Edge cases: optional description, timestamp serialization, and future filter parameters

## Dependencies
- Empty repo scaffold and no existing app conventions
- Chosen frontend and backend package managers for the new apps
- Basic testing tools available after scaffold creation

## Validation
- Confirm the backend app imports cleanly and can start in scaffold form
- Confirm the frontend app builds or type-checks in scaffold form
- Confirm issue contract names and enum literals match on both sides
- Record any setup constraints that will matter for Pass 2

## Expected implementer feedback
The implementer must report back using this exact structure:
- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

The implementer must also state whether the scaffold and tooling choices matched the planning assumptions or need adjustment.

## Stop after this pass
Do not continue to persistence, real API behavior, or frontend data flows.
Stop after Pass 1 and wait for architect review and the next instruction.

# Pass 2 - Backend Issue API, Validation, and SQLite

**Status:** Completed

## Objective
Implement the backend issue API, request validation, timestamps, and SQLite persistence.

## In scope
- GET /issues
- GET /issues/{id}
- POST /issues
- PUT /issues/{id}
- Optional status filtering for the issue list
- SQLite schema and data access
- Validation for invalid payloads
- Timestamp handling
- Backend dependency/bootstrap setup needed to run the pass validation

## Out of scope
- Frontend implementation beyond any contract adjustments required by backend reality
- Major schema redesigns
- Unrelated refactors
- Frontend dependency installation as a Pass 2 task unless it is required to preserve a repo convention discovered during the pass

## Architecture requirements
- Keep JSON payloads and error semantics stable
- Favor explicit validation and small data-access helpers
- Keep the SQLite setup simple and local
- Preserve the contract established in Pass 1
- Do not alter the issue field set or enum values

## Suggested structure
- Backend model, schema, repository, and router modules
- Backend test files that exercise the API and persistence layer

## Functional flow
- Before coding, confirm or install the backend validation dependencies needed to run tests
- Request enters FastAPI, validates payload, reads or writes SQLite, and returns a JSON response
- The issue list may accept a status filter without changing the underlying contract

## Requirements by subtask/module

### Issue API
- Responsibility: serve list, detail, create, and update operations
- Inputs: filter query parameters, issue IDs, and issue payloads
- Outputs: issue JSON responses or validation and not-found errors
- Expected behavior: the API remains small and predictable
- Error handling: invalid payloads return clear 4xx responses and missing records return 404s
- Edge cases: empty result sets, duplicate IDs, unchanged updates, and invalid status filters

### SQLite persistence
- Responsibility: store issues locally and keep timestamps current
- Inputs: validated issue payloads
- Outputs: persisted rows that match the contract
- Expected behavior: persisted data survives process restarts
- Error handling: database errors are surfaced cleanly and do not corrupt the contract
- Edge cases: empty database, first-run schema creation, and text fields with empty description

### Backend validation
- Responsibility: reject invalid payloads before persistence
- Inputs: create and update requests
- Outputs: structured validation errors
- Expected behavior: required fields are enforced and enum values are checked
- Error handling: invalid status, invalid priority, missing title, and empty title are rejected
- Edge cases: whitespace-only titles, missing description, malformed JSON, and status values outside the contract

### Bootstrap and test environment
- Responsibility: make validation runnable in the current workspace
- Inputs: the existing Python environment and repository tooling
- Outputs: an executable backend test setup with `pytest` and `httpx` available
- Expected behavior: the implementer resolves backend test dependencies before declaring the pass complete
- Error handling: if dependencies cannot be installed, report the exact blocker and continue only with the code work that can be validated
- Edge cases: missing package manager commands, locked environments, and workspace write restrictions

## Dependencies
- Pass 1 scaffold
- Backend app and schema modules created in Pass 1
- Backend validation environment with `pytest` and `httpx` available before test execution

## Validation
- Backend automated tests for list, detail, create, update, and invalid payloads
- Smoke check that the app starts against SQLite
- Verify timestamps and status filtering behavior
- Confirm the backend test environment was bootstrapped if it was missing at the start of the pass

## Expected implementer feedback
The implementer must report back using this exact structure:
- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

The implementer must also state whether backend bootstrap was required, whether it was completed, and whether any frontend dependency gap remains pending for Pass 3.

## Stop after this pass
Stop after backend API and persistence are complete.
Do not continue into frontend read flow work.

# Pass 3 - Frontend Read Flow

**Status:** Completed

## Objective
Render the issue list, filter by status, and surface loading, empty, success, and error states against the backend API completed in Pass 2.

## In scope
- Issue list page wired to `GET /issues`
- Status filter control using the existing backend status query support
- Read-only data fetching and refresh on filter changes
- Loading, empty, success, and error UI states
- Minimum frontend coverage needed to lock the read-flow behavior

## Out of scope
- Create, edit, and mark-done mutations
- Backend contract changes unless a real mismatch is discovered while wiring the read flow
- Broad frontend refactors
- Mutation-specific test coverage
- New product features or pagination

## Architecture requirements
- Keep state updates simple and predictable
- Use the backend contract as-is
- Keep the filter flow server-driven instead of inventing a second client-side contract
- Keep the UI shell and state components small enough to extend in Pass 4

## Suggested structure
- Frontend page, query helper, list component, filter component, and async-state components
- Test files or test harness additions only if needed to validate the read flow
- Reuse the existing `frontend/src/types`, `frontend/src/api`, and `frontend/src/components` layout

## Functional flow
- Pass 3 begins by bootstrapping the frontend validation environment if `node_modules` or the required scripts are missing
- The page loads, fetches issues from `GET /issues`, and forwards the selected status as a query parameter
- The UI renders loading first, then success, empty, or error states depending on the response
- Changing the filter triggers a new read request and updates the rendered list without touching mutation flows

## Requirements by subtask/module

### Issue list page
- Responsibility: display the issue collection and connect it to the current filter state
- Inputs: issue data, the selected status filter, and request state
- Outputs: rendered issue rows, empty-state text, or error messaging
- Expected behavior: the list reflects the current backend response and updates when the filter changes
- Error handling: fetch failures remain visible instead of collapsing into an empty state
- Edge cases: no issues, long titles, and mixed statuses

### Status filter
- Responsibility: change the visible issue subset
- Inputs: the selected status value
- Outputs: updated query state and a new read request
- Expected behavior: the control is simple, deterministic, and uses the existing backend status values
- Error handling: unsupported filter values fall back safely to the default view
- Edge cases: repeated selection of the same status and no match after filtering

### Read states
- Responsibility: show loading, empty, success, and error states
- Inputs: async request lifecycle events
- Outputs: deterministic UI states
- Expected behavior: each state is easy to recognize and does not flicker unnecessarily
- Error handling: network failures and empty results are handled distinctly
- Edge cases: slow response, empty list, and recovery after failure

### Bootstrap and validation environment
- Responsibility: make frontend validation runnable in the current workspace
- Inputs: the existing Node/npm environment and repository tooling
- Outputs: installed frontend dependencies and a runnable frontend validation setup
- Expected behavior: the implementer installs dependencies at the start of the pass if required to run `npm run typecheck`, `npm run build`, or the small frontend coverage
- Error handling: if dependencies cannot be installed or the workspace blocks runtime writes, report the exact blocker and continue only with the parts that can be validated
- Edge cases: missing package manager lockfile, unavailable npm scripts, and sandbox restrictions

## Dependencies
- Pass 2 backend API and status filtering
- Frontend package metadata from Pass 1
- Frontend dependency installation at the start of the pass if needed to run validation

## Validation
- Small frontend coverage for loading, empty, error, and success behavior
- `npm run typecheck` and/or `npm run build` after bootstrap
- Confirm the filter changes the rendered issue list
- Confirm the frontend environment was bootstrapped if dependencies were missing

## Expected implementer feedback
The implementer must report back using this exact structure:
- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

The implementer must also state whether frontend bootstrap was required, whether it was completed, and whether the pass stayed within the read-flow scope.

## Stop after this pass
Do not continue to create/edit/done mutations.
Stop after Pass 3 and wait for architect review and the next instruction.

# Pass 4 - Frontend Write and Update Flow

**Status:** Completed

## Objective
Add create, edit, and mark-done interactions while reusing the existing right-side form shell for both create and edit mode.

## In scope
- Create new issue form
- Edit existing issue form in the current form shell
- Mark an issue as done
- Form validation and server validation display
- Refresh the list after successful mutations
- Keep the read flow intact while adding mutation handling
- Keep frontend validation coverage focused on mutation happy paths and error display

## Validation
- Targeted frontend coverage for mutation happy paths and error display
- Smoke check that create, edit, and mark-done flows update the list
- `npm run typecheck` and `npm run build` after any required frontend bootstrap
- Run frontend tests after bootstrap; if the workspace again blocks test runner spawn or other runtime writes, report that explicitly and use escalation as needed to complete validation

## Out of scope
- Backend API changes unless a real contract mismatch is discovered
- A separate edit page, modal, or drawer
- Pagination, authentication, file uploads, websockets, and background jobs
- Broad frontend refactors unrelated to mutation wiring

## Architecture requirements
- Keep the mutation flow explicit, small, and easy to test
- Preserve the existing issue contract and status semantics
- Reuse the current right-side form shell for create and edit mode instead of branching into a second layout
- Prefer refresh-after-success rather than optimistic list updates
- Do not add abstractions that later passes must unwind

## Suggested structure
- Frontend form behavior in `frontend/src/components/IssueFormShell.tsx`
- Mutation-aware page logic in `frontend/src/pages/IssueTrackerPage.tsx`
- API helpers and shared types in the existing frontend `src/api` and `src/types` layout
- Focused frontend tests alongside the affected page or component files

## Functional flow
- The user selects create or edit from the current workspace
- The existing right-side form shell swaps between create and edit mode in place
- Submitting the form sends the mutation request, surfaces validation or server errors, then refreshes the issue list on success
- Mark done updates the selected issue to `done` and refreshes the list using the same read path as Pass 3

## Requirements by subtask/module

### Create/edit form
- Responsibility: collect issue fields and submit create or update payloads
- Inputs: title, description, status, priority, and the selected issue when editing
- Outputs: create or update requests, visible validation errors, and refreshed issue data on success
- Expected behavior: the same shell supports both create and edit with minimal branching
- Error handling: required fields, server validation errors, and mutation failures remain visible
- Edge cases: empty description, whitespace-only title, unchanged edits, and preserving the current form when validation fails

### Mark-done action
- Responsibility: update an issue status to `done`
- Inputs: the issue row or issue record selected from the list
- Outputs: persisted `done` state and a refreshed issue list
- Expected behavior: the action is simple and visible from the list
- Error handling: failed mutation shows a user-facing error state
- Edge cases: already-done issues, stale data after refresh, and repeated clicks

### Mutation feedback
- Responsibility: surface success and failure after create, edit, or done actions
- Inputs: mutation result, validation error, or network error
- Outputs: refreshed issue list and visible status feedback
- Expected behavior: the UI shows the effect of the change without hidden transitions
- Error handling: network errors and validation failures remain visible
- Edge cases: submission retries and preserving user input after a validation error

### Bootstrap and validation environment
- Responsibility: make frontend mutation validation runnable in the current workspace
- Inputs: the existing Node/npm environment and repository tooling
- Outputs: installed or verified frontend dependencies plus runnable type-check, build, and test commands
- Expected behavior: if dependencies or scripts are missing, the implementer bootstraps them before declaring the pass complete
- Error handling: if runtime writes or test runner spawn are blocked again, report the exact blocker and use escalation if needed for validation
- Edge cases: missing lockfile drift, unavailable npm scripts, and sandbox restrictions

## Dependencies
- Pass 3 frontend read flow
- The existing right-side form shell from the current frontend scaffold
- Frontend dependency installation if validation cannot run with the current local toolchain

## Expected implementer feedback
The implementer must report back using this exact structure:
- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

The implementer must also state whether frontend bootstrap was required, whether it was completed, whether the pass stayed within mutation-flow scope, and whether the edit UX reused the existing form shell as instructed.

## Stop after this pass
Stop after Pass 4 and wait for architect review and the next instruction.

# Pass 5 - Tests, Cleanup, and Final Review

**Status:** Completed

## Objective
Harden the backend with the remaining edge-case tests, remove temporary scaffolding that is no longer needed, and prepare the final codebase for reviewer audit. Keep frontend changes to a minimum and only add a targeted regression test if a real gap appears while closing out the pass.

## In scope
- Expand backend test coverage for validation and persistence edge cases
- Add at most one narrowly targeted frontend regression test only if a real gap is discovered during final cleanup
- Remove temporary scaffolding that is no longer needed
- Confirm the final plan and implementation line up
- Run the final relevant test set and prepare for reviewer audit

## Out of scope
- New features
- API or contract changes
- Broad frontend refactors
- Reworking the existing read or mutation flows
- Extra frontend edge-case work that is not needed to close a real gap

## Architecture requirements
- Keep the codebase small and readable
- Do not add cleanup that changes behavior
- Preserve the existing issue contract, route surface, and UI flow
- Prefer the narrowest test additions that meaningfully increase confidence
- Do not widen frontend scope unless a specific regression gap is found

## Suggested structure
- Backend test files alongside the issue API and repository code
- Minimal frontend regression coverage only if it closes a concrete final-pass gap
- Cleanup limited to the files already touched by the staged workflow

## Functional flow
- Review the current backend and frontend state
- Add or tighten only the tests needed to prove the final edge cases
- Remove temporary scaffold artifacts that are no longer needed
- Run the final validation commands
- Hand the repo off to reviewer audit with no remaining implementation work

## Requirements by subtask/module

### Backend edge-case coverage
- Responsibility: prove the issue API and persistence behavior on the less common paths
- Inputs: representative requests, invalid payloads, and SQLite states
- Outputs: passing automated tests that cover the real backend edge cases
- Expected behavior: invalid payloads, missing records, empty database states, and timestamp behavior remain correct
- Error handling: assert explicit errors for malformed or invalid requests
- Edge cases: whitespace-only titles, invalid enums, empty update payloads, missing issue IDs, and empty result sets

### Frontend regression check
- Responsibility: catch any final UI regression that would not already be covered by Pass 3 and Pass 4
- Inputs: current page state and any discovered gap in the mutation or read flows
- Outputs: at most one focused test that protects the discovered gap
- Expected behavior: only add this work if it meaningfully tightens confidence
- Error handling: validate visible failure behavior only if the gap is real
- Edge cases: edit-to-create reset, refresh-after-success, or validation-error display only if one of these is not already well covered

### Cleanup and final review prep
- Responsibility: remove temporary scaffold pieces and leave the repo ready for the reviewer
- Inputs: the completed staged implementation
- Outputs: a tidy final codebase and current plan notes
- Expected behavior: no unnecessary scaffolding remains and the plan still matches the implementation
- Error handling: cleanup must not alter behavior
- Edge cases: keep any helpers that are still needed for tests or app startup

## Dependencies
- Pass 4 frontend mutation flow
- Pass 2 backend API and SQLite persistence
- Existing test tooling and package setup from the prior passes

## Validation
- Full backend test suite, including the tightened edge-case coverage
- Targeted frontend tests only if a real regression gap is found
- Final smoke check of the app flows
- Reviewer audit against the plan

## Expected implementer feedback
The implementer must report back using this exact structure:
- Pass
- Status
- Files changed
- What was implemented
- Validation performed
- Findings
- Issues / blockers
- Proposed adjustments for architect review
- Decision needed before next pass

The implementer must also state whether any final bootstrap was required, whether the narrowed frontend regression check was needed, whether the pass stayed within cleanup/test scope, and whether any temporary scaffold pieces were removed or retained.

## Stop after this pass
Stop after Pass 5 and wait for reviewer audit.
