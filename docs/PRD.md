# AssetsBoard Product Requirements Document

AssetsBoard is a web application for private investors to track assets, update positions, and review portfolio value summaries with lightweight local persistence.

## Product Vision

Provide a fast, low-friction portfolio board where an individual investor can add, update, and review holdings in minutes, while creating a clear path from local-first prototype behavior to a production-grade backend platform.

## Users and Personas

### Persona 1: Private Investor (Primary)
- Profile: Individual managing a personal portfolio across categories (crypto, real estate, commodities, stocks, currencies).
- Needs: Quick add/edit flows, clear portfolio totals, simple lookup of supported symbols.
- Current fit: Strong for single-device usage and exploratory tracking.

### Persona 2: Workshop Learner / Maintainer (Secondary)
- Profile: Engineer learning or maintaining a legacy-style Angular app.
- Needs: Understandable flows, deterministic local behavior, testable user journeys.
- Current fit: Strong via modular routes, repositories, and Cypress coverage.

## Problem Statement

Private investors need a simple way to maintain a categorized investment snapshot without spreadsheet overhead. The current solution works well for local, single-user tracking but lacks multi-user identity, durable backend storage, and synchronization across devices.

## Goals

1. Keep add/update/delete asset workflows reliable and fast.
2. Keep summary visibility clear at both row and portfolio-total levels.
3. Keep symbol search usable with URL-driven filtering.
4. Preserve theme preference and asset state between sessions on the same device.
5. Prepare data contracts and architecture for backend persistence and authentication.

## Non-Goals (Current Phase)

1. No multi-user account system in the running app.
2. No server-side persistence in the running app.
3. No portfolio analytics beyond current totals and list views.
4. No broker/API integrations.

## Functional Requirements

### FR-1 Add Asset
- Users can create a new asset from /assets/new by selecting category and symbol, then entering quantity.
- Created asset is persisted to browser localStorage and appears in the home list.
- Status: Implemented

### FR-2 Update Existing Asset
- Users can open an asset-specific edit route and update values (for example quantity), then save.
- Updated asset is persisted to localStorage and reflected in the portfolio list.
- Status: Implemented

### FR-3 Delete Asset
- Users can delete an asset from the edit flow.
- Deleted asset is removed from store state and localStorage, then user is returned to home.
- Status: Implemented

### FR-4 View Portfolio Summary
- Users can view a tabular summary with asset, category, quantity, value, and amount.
- Header shows portfolio total amount derived from current in-memory state.
- Status: Implemented

### FR-5 Search Symbols
- Users can browse/search supported symbols on /symbols.
- Search updates URL query params and page can be initialized from existing query param.
- Status: Implemented

### FR-6 Theme Toggle and Preference
- Users can toggle light/dark theme from the header.
- Theme is applied to the document root and persisted in localStorage.
- Status: Implemented

### FR-7 Transition to Backend Persistence
- System shall support replacing in-memory/localStorage repositories with API-backed repositories while preserving route and component contracts.
- Initial target capabilities: authenticated user context, CRUD assets, category/symbol catalogs.
- Status: NotStarted

## Non-Functional Requirements

1. Responsiveness: UI must remain usable on desktop and mobile form factors.
2. Performance: common route interactions (home, symbols search, form submit) should feel near-instant in normal browser conditions.
3. Reliability: local persistence operations must not corrupt the stored asset list under normal usage.
4. Testability: core journeys (home, new asset, edit asset, symbols, theme) must stay covered by automated E2E tests.
5. Maintainability: new features should follow existing route/module and service boundaries to limit regression risk in legacy code.

## Constraints and Assumptions

### Constraints
- Brownfield Angular 15 architecture and existing module/route structure must be preserved.
- Current data source is frontend-heavy (fake repository + localStorage), not a live backend.
- Existing E2E stack is Cypress (despite older docs mentioning Playwright).
- Existing state pattern uses custom store/effects services, not NgRx.

### Assumptions
- Single-user, single-browser context is acceptable for current production/workshop usage.
- Symbols and categories are finite and maintained in static/front-end repositories for now.
- Temporary market-value refresh logic remains best-effort and non-auditable.

## Success Metrics

1. Asset lifecycle success rate: at least 99% successful add/update/delete flows in QA and smoke validation.
2. E2E stability: core Cypress suite pass rate at or above 95% on CI over rolling 30 days.
3. Task completion time: users can add a new asset in under 30 seconds median during usability checks.
4. Regression trend: no net increase in high-severity defects for portfolio CRUD journeys across releases.

## Release Scope

### MVP (Current Brownfield Baseline)
- Home summary list and total amount display.
- Add asset flow.
- Edit and delete asset flow.
- Symbols browsing and search with query param support.
- Theme toggle persisted in localStorage.

### Next Release (Backend Evolution)
- Introduce backend service for assets/categories/symbols with repository abstraction preserved.
- Add user identity/authentication layer and user-bound asset ownership.
- Define migration strategy from localStorage state to server-backed records.
- Add API contract tests and progressively shift E2E from local-only assumptions to integrated flows.

## Risks

1. State mutation duplication risk in add flow (optimistic local mutation plus effect-driven mutation) may cause double insert behavior in future refactors.
2. LocalStorage-only persistence risks data loss on browser clearing and prevents cross-device continuity.
3. Manual subscriptions in effects and route logic can create maintenance or lifecycle risks as complexity grows.
4. Documentation drift between intended target architecture (backend/JWT/SQLite) and current implementation can cause planning errors.

## Open Questions

1. What backend stack is the committed target for delivery (Bun/TS API as in briefing or another platform)?
2. What is the required authentication timeline: pre-backend, backend MVP, or post-MVP?
3. Should historical valuation snapshots be stored, or only current holdings state?
4. What data migration UX is expected for existing localStorage users when server persistence arrives?
5. Should Cypress remain the long-term E2E standard, or should the project transition to Playwright to match earlier documentation?
