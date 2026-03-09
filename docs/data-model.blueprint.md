---
title: Legacy Angular AI Workshop - Where To Change What
purpose: Provide a practical edit map that tells engineers and AI agents exactly where to modify code for common feature requests in this project.
prompt_template: |
  Build a "where-to-change-what" implementation map for this Angular codebase.
  For each common request, provide:
  1) files to edit,
  2) expected side effects,
  3) tests to run,
  4) common pitfalls in legacy patterns.
  Cover routes, forms, state, repositories, and UI shell.
target_audience: Workshop participants applying AI-assisted maintenance to legacy Angular apps.
model_hint: GPT-5.3-Codex
last_updated: 2026-03-09
---

# Where To Change What

## Add a new field to Asset (example: acquisitionDate)

Edit these files:
1. `assets-board/src/app/domain/asset.type.ts`
2. `assets-board/src/app/routes/assets/new/new-asset-form/new-asset-form.component.ts`
3. `assets-board/src/app/routes/assets/new/new-asset-form/new-asset-form.component.html`
4. `assets-board/src/app/routes/assets/edit/edit-asset-form/edit-asset-form.component.ts`
5. `assets-board/src/app/routes/assets/edit/edit-asset-form/edit-asset-form.component.html`
6. `assets-board/src/app/shared/assets/assets-repository.service.ts`
7. `tests/cypress/e2e/new-asset.cy.js` and `tests/cypress/e2e/edit-assets.cy.js`

Side effects to verify:
1. LocalStorage serialization and hydration.
2. Display in home list if needed.
3. Existing fixtures still parse.

## Add a new asset category

Edit these files:
1. `assets-board/src/app/shared/categories-repository.service.ts`
2. `assets-board/src/app/shared/symbols-repository.service.ts`
3. `assets-board/src/app/shared/assets/asset-value.service.ts`
4. `assets-board/src/app/shared/assets/asset-details.service.ts`
5. Potentially `assets-board/src/app/domain/*.type.ts` for enum/type extension.

Side effects to verify:
1. New category appears in new asset form.
2. Symbols list includes that category where expected.
3. Edit details resolve without null regressions.

## Change create/update/delete behavior

Edit these files:
1. `assets-board/src/app/routes/assets/new/new-asset.service.ts`
2. `assets-board/src/app/routes/assets/edit/edit-asset.service.ts`
3. `assets-board/src/app/shared/assets/assets-store.service.ts`
4. `assets-board/src/app/shared/assets/asset-effects.service.ts`
5. `assets-board/src/app/shared/assets/assets-repository.service.ts`

Legacy pitfall:
1. New flow currently does optimistic store mutation and effect-driven mutation.
2. If you change one path, align the other to avoid duplicate inserts.

## Update validations in forms

Edit these files:
1. `assets-board/src/app/shared/custom.validations.ts`
2. `assets-board/src/app/routes/assets/new/new-asset-form/new-asset-form.component.ts`
3. Optional error messaging in `assets-board/src/app/routes/assets/new/new-asset-form/new-asset-form.component.html`

Run:
1. Manual check for invalid states.
2. E2E new asset flow in `tests/cypress/e2e/new-asset.cy.js`.

## Modify symbols search behavior

Edit these files:
1. `assets-board/src/app/routes/symbols/symbols.component.ts`
2. `assets-board/src/app/shared/symbols-repository.service.ts`
3. `assets-board/src/app/routes/symbols/symbols.component.html`
4. `tests/cypress/e2e/symbols.cy.js`

Side effects to verify:
1. Query param sync still works.
2. Search term preload from URL still initializes input.

## Change header total amount or navigation

Edit these files:
1. `assets-board/src/app/core/layout/header/header.component.ts`
2. `assets-board/src/app/core/layout/header/header.component.html`
3. `assets-board/src/app/shared/assets/assets-store.service.ts` if total math changes.

Run:
1. Home smoke test.
2. Quick manual check after add/update/delete.

## Theme behavior changes

Edit these files:
1. `assets-board/src/app/core/layout/theme-toggle/theme-toggle.component.ts`
2. `assets-board/src/app/core/layout/theme-toggle/theme-toggle.component.html`
3. `assets-board/src/styles.css`
4. `tests/cypress/e2e/theme.cy.js`

Side effects to verify:
1. Theme persisted in localStorage.
2. Initial page load reflects stored theme.
