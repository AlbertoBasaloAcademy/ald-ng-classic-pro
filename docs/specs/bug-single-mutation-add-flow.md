# bug-single-mutation-add-flow Specification
- Type: bug
- Status: Draft
- Feature Status: InProgress
- Owner: Product
- Date: 2026-03-09

## Problem
The current add-asset flow has a duplicate insert risk because state can be mutated in more than one step of the same user command. In the baseline architecture, the add sequence includes an optimistic local reducer update and an effect-driven reducer update tied to repository persistence. This creates non-deterministic behavior under refactor, retry, or asynchronous timing changes, and can produce duplicate rows and incorrect portfolio totals.

## Scope
- Enforce a single, canonical mutation path for add-asset state updates.
- Define sequencing rules for add command handling from form submit to persisted state update.
- Define tests that prove exactly one insert occurs per add command.
- Cover unit and E2E acceptance behavior for duplicate prevention in add flow.
- Keep this change independent from identity migration and backend transition work.

## Non-Scope
- No redesign of edit or delete flows.
- No migration from symbol-based selection to id-based update/delete in this item.
- No backend API implementation or auth context changes.
- No broader store framework migration (for example NgRx adoption).
- No UI redesign of add-asset page.

## User Impact
- Users can add an asset once and reliably see one resulting row.
- Portfolio totals remain accurate after add operations.
- Repeated save triggers or timing variance do not create duplicate inserts.
- Behavior is predictable for maintainers and lowers regression risk in future persistence changes.

## Solution Outline
### Flow Sequencing
- Treat add as a single command with one authoritative state-mutation step.
- Sequence: submit form -> dispatch add command -> run persistence side effect -> apply one reducer insert from the authoritative result.
- Remove or disable any additional optimistic reducer insert in the add flow path.
- Keep failure path explicit so failed persistence does not create a phantom inserted row.

### State and Logic Rules
- One add command maps to at most one reducer insert mutation.
- Duplicate insert on the same command lifecycle is not allowed.
- Add flow remains deterministic across async execution ordering.

### Tests Strategy
- Add unit tests for store/effects interaction to verify single reducer insert invocation per add command.
- Add guard tests for retry or duplicate event dispatch scenarios so final state still has one inserted record for one user intent.
- Add or update Cypress add-flow test to verify one new row appears and totals reflect one insert.

## Affected Files and Modules
- Module: Add asset route and submit orchestration
  - assets-board/src/app/routes/assets/new/
- Module: Asset store action and reducer path
  - assets-board/src/app/shared/assets/assets-store.service.ts
  - assets-board/src/app/shared/assets/assets-actions.type.ts
- Module: Effect orchestration and persistence side effects
  - assets-board/src/app/shared/assets/asset-effects.service.ts
  - assets-board/src/app/shared/assets/assets-repository.service.ts
- Tests: Unit and E2E regression coverage
  - assets-board/src/app/**.spec.ts
  - tests/cypress/e2e/new-asset.cy.js

## Acceptance Criteria
- [ ] WHEN a user submits add-asset once THE System SHALL persist and apply exactly one inserted asset in state.
- [ ] WHEN add-asset processing completes successfully THE System SHALL have exactly one reducer insert mutation for that command lifecycle.
- [ ] IF add-asset persistence fails THEN THE System SHALL not leave a newly inserted phantom asset in state.
- [ ] WHEN add-asset succeeds THE System SHALL display exactly one newly added row in the home asset list.
- [ ] WHEN add-asset succeeds THE System SHALL update portfolio total amount exactly once for that inserted asset.
- [ ] WHERE add-flow unit tests run THE System SHALL include automated assertions that detect duplicate insert mutations.
- [ ] WHERE add-flow E2E tests run THE System SHALL fail if more than one row is added for one submit action.
- [ ] WHILE current localStorage persistence is active THE System SHALL preserve existing add/edit/delete user-visible behavior except duplicate prevention.

## Risks
- Removing optimistic insert may change perceived immediacy of add feedback.
- Refactor in effect/reducer wiring can introduce missed insert if sequencing is implemented incorrectly.
- Existing tests may be coupled to prior mutation order and require careful updates.

## Rollout Plan
1. Implement single mutation path behind current add command contract.
2. Add or update unit tests for reducer/effects single-insert guarantees.
3. Add or update Cypress new-asset scenario for duplicate prevention.
4. Run full relevant test suites (Angular unit + Cypress add flow).
5. Release in one increment with changelog note under bug fix.
6. Monitor regression signals in CRUD flows during next QA cycle.

## Implementation Tracking Checklist
- [ ] Confirm canonical mutation path location and remove secondary insert path.
- [ ] Update add-flow orchestration logic and keep deterministic sequence.
- [ ] Add unit tests for single insert and failure behavior.
- [ ] Update Cypress add-flow test for one-row and one-total-change verification.
- [ ] Validate no regression in edit/delete/theme/symbol smoke checks.
- [ ] Mark spec status transitions: Draft -> Planned -> Coded -> Verified -> Released.
