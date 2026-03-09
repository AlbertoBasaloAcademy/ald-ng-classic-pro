# Spec: Migrate Asset CRUD Identity to ID-Based Operations

## Metadata
- Spec ID: `feat-id-based-asset-crud`
- Type: `feat`
- Status: `Draft`
- Feature Status: `InProgress`
- Date: 2026-03-09
- Owner: Product/Engineering

## Problem
Asset update and delete flows currently rely on `symbol` in key command paths, while the domain and target architecture define `id` as the canonical asset identity. This mismatch creates ambiguity when symbols are reused, renamed, or mapped across categories/providers, and increases migration risk for backend persistence where records are addressed by immutable IDs.

## Scope
- Define and adopt `assetId` as the canonical identity for all asset write commands (update and delete).
- Standardize command payload shapes so write operations use explicit id-based contracts.
- Align route, store action payload, effect input, and repository method contracts around id-based addressing.
- Preserve current user-visible CRUD behavior while changing internal identity semantics.

## Non-Scope
- No redesign of UI layout or navigation patterns.
- No backend API implementation or auth implementation.
- No change to quote calculation, totals logic, or symbols search behavior.
- No broad state management rewrite beyond identity/payload contract alignment.

## User Impact
- Users continue to create, edit, and delete assets with the same screens and expected outcomes.
- Reliability improves in edge cases where symbol-based targeting could affect the wrong record.
- Existing local assets remain usable after migration with deterministic id targeting.

## Solution Outline

### Identity Model
- Declare `id` as the only write identity for asset mutation commands.
- Treat `symbol` as a descriptive/business field, not a mutation key.

### Command Payload Contracts
- Replace symbol-targeted write payloads with explicit id-targeted payloads.
- Preferred command contract shapes:
  - Update: `{ id, changes }`
  - Delete: `{ id }`
- Ensure payload validation/guards fail fast when `id` is missing or invalid.

### Route and Selection Contract
- Edit/delete flows resolve a selected asset by `id`.
- Routing parameters for edit/delete paths use `id`-oriented resolution.

### Store/Effects/Repository Contract Alignment
- Store actions for update/delete carry id-based payloads only.
- Effects pass id-based payloads to repositories and reducers.
- Repository update/delete operations target records by id.
- Keep reducer mutations deterministic with a single selected record per id.

### Data Compatibility
- Maintain compatibility with existing local data by ensuring each stored asset has an id.
- If legacy records are missing id, assign deterministic ids once before write operations proceed.

## Affected Files/Modules
- Domain contracts:
  - `assets-board/src/app/domain/asset.type.ts`
- Asset route modules and edit/new flows:
  - `assets-board/src/app/routes/assets/assets-routing.module.ts`
  - `assets-board/src/app/routes/assets/edit/**`
  - `assets-board/src/app/routes/assets/new/**`
- Asset state and side effects:
  - `assets-board/src/app/shared/assets/assets-actions.type.ts`
  - `assets-board/src/app/shared/assets/assets-store.service.ts`
  - `assets-board/src/app/shared/assets/asset-effects.service.ts`
- Asset repository and write services:
  - `assets-board/src/app/shared/assets/assets-repository.service.ts`
  - `assets-board/src/app/shared/assets/asset-details.service.ts`
- Verification coverage:
  - `assets-board/src/**/*.spec.ts`
  - `tests/cypress/e2e/edit-assets.cy.js`
  - `tests/cypress/e2e/new-asset.cy.js`

## Acceptance Criteria (Testable)
1. When an update command is dispatched, the system shall require an `id` in the payload and shall not rely on `symbol` for record targeting.
2. When a delete command is dispatched, the system shall require an `id` in the payload and shall not rely on `symbol` for record targeting.
3. When an edit route is opened for an existing asset, the selected asset shall be resolved by `id`, and saving shall update exactly one asset with that id.
4. When a delete action is confirmed from the edit flow, exactly one asset with the requested id shall be removed from state and persistence.
5. If two assets share the same symbol, update/delete operations shall affect only the asset whose id is provided.
6. If a write command is submitted without a valid id, the system shall reject the command and shall leave state and persistence unchanged.
7. Existing user-visible CRUD flows shall remain functional (create, edit, delete) with no regression in expected navigation outcomes.
8. Automated unit and E2E tests covering asset edit/delete shall pass with id-based payload expectations.

## Risks
- Legacy local records may have unstable or missing ids, causing inconsistent targeting during migration.
- Partial migration (actions updated but repository not updated) may break write operations.
- Existing tests may encode symbol-based assumptions and fail until updated.
- Route param changes may introduce broken deep links if not handled with compatibility mapping.

## Rollout Plan
1. Define final id-based command payload contracts for update/delete.
2. Update route param handling and selected-asset resolution to id.
3. Update store actions, effects, and repository methods to id-based targeting.
4. Add compatibility guard for legacy local records missing id.
5. Update unit and Cypress tests to assert id-based write targeting.
6. Validate manual CRUD flows and run full automated suites.
7. Release behind normal QA gate; monitor for edit/delete regressions.

## Status and Checklist

### Current Status
- Spec: `Draft`
- Feature: `InProgress`

### Implementation Checklist
- [ ] Confirm final action payload contracts for update/delete are id-only.
- [ ] Update edit/delete routing and selected-asset lookup to id.
- [ ] Refactor repository update/delete targeting to id.
- [ ] Add legacy id backfill/guard for local records.
- [ ] Update unit tests for actions/effects/reducers/repository behavior.
- [ ] Update Cypress tests for edit/delete identity behavior.
- [ ] Run app test suite and Cypress suite successfully.
- [ ] Validate no CRUD UX regression in manual smoke checks.
