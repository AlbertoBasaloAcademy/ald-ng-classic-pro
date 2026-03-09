---
title: Legacy Angular AI Workshop - 5 Minute Onboarding Path
purpose: Provide a fast, repeatable onboarding guide for engineers and AI agents joining this Angular 15 legacy-style project.
prompt_template: |
  Analyze this Angular workspace and generate a "5-minute onboarding path" for a new engineer.
  Include:
  1) exact file reading order,
  2) runtime flow from bootstrap to first rendered page,
  3) where state and side effects are managed,
  4) where tests live and how to run them,
  5) top legacy risks to watch in first edits.
  Keep it practical and map each step to concrete files.
target_audience: Workshop participants learning to apply AI to legacy Angular projects.
model_hint: GPT-5.3-Codex
last_updated: 2026-03-09
---

# 5 Minute Onboarding Path

## Minute 1 - Boot and shell

1. Read `assets-board/src/main.ts` to see app bootstrap.
2. Read `assets-board/src/app/app.module.ts` for root imports.
3. Read `assets-board/src/app/app.component.html` for shell composition.

Outcome: you know how Angular starts and where routed content is mounted.

## Minute 2 - Route map

1. Read `assets-board/src/app/app-routing.module.ts`.
2. Identify lazy entry points:
   - home (`/`)
   - assets (`/assets/new`, `/assets/edit/:symbol`)
   - symbols (`/symbols`)

Outcome: you know feature boundaries and likely impact areas for changes.

## Minute 3 - State and side effects

1. Read `assets-board/src/app/shared/assets/assets-store.service.ts`.
2. Read `assets-board/src/app/shared/assets/asset-effects.service.ts`.
3. Read `assets-board/src/app/shared/assets/assets-repository.service.ts`.

Outcome: you understand the custom store pattern:
- actions are dispatched manually,
- effects subscribe and call repositories,
- reducers are imperative methods on the store service,
- persistence is localStorage plus in-memory fake data.

## Minute 4 - User journeys

1. Home list:
   - `assets-board/src/app/routes/home/home.component.ts`
   - `assets-board/src/app/routes/home/assets-list/assets-list.component.html`
2. New asset:
   - `assets-board/src/app/routes/assets/new/new.component.ts`
   - `assets-board/src/app/routes/assets/new/new-asset-form/new-asset-form.component.ts`
3. Edit asset:
   - `assets-board/src/app/routes/assets/edit/edit.component.ts`
   - `assets-board/src/app/routes/assets/edit/edit-asset-form/edit-asset-form.component.ts`

Outcome: you can trace create, update, and delete behavior from UI to state.

## Minute 5 - Tests and safe-first-change strategy

1. Unit test command: `cd assets-board && npm test`
2. E2E tests live in `tests/cypress/e2e`.
3. E2E command:
   - open mode: `cd tests && npm start`
   - headless mode: `cd tests && npm test`

Safe-first-change checklist:
1. Prefer scoped changes in one route or one shared service.
2. Validate localStorage behavior for assets and theme.
3. Run at least one happy-path e2e related to your change.

## Legacy risks to highlight in workshops

1. Manual subscriptions in effects without teardown strategy.
2. Store methods and effect dispatch can both mutate state in add flow.
3. Fake repositories can hide integration assumptions.
4. Validation and form behavior are split between template and component logic.
