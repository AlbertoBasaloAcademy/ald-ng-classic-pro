# AssetsBoard Architecture Design Document (ADD)

## Table of Contents
1. Scope and Inputs
2. Brownfield Baseline
3. Target Architecture
4. C4-lite View
5. Key Flows
6. ADRs
7. Risks and Mitigations
8. Observability, Testing, and Deployment
9. File-Path Mapping
10. Delivery Roadmap

## 1) Scope and Inputs

This ADD uses `docs/PRD.md` as source of truth and aligns with the implemented Angular app.

In scope:
- Brownfield architecture for current app in `assets-board/`.
- Target evolution to backend persistence and authenticated ownership.
- Data model transition guidance that matches `docs/ERD.md`.

Out of scope:
- Full backend implementation details beyond architecture boundaries.
- Broker integrations and advanced analytics (explicit PRD non-goals).

## 2) Brownfield Baseline (Current)

### Runtime and topology
- Single SPA frontend (Angular 15) served on `http://localhost:4200`.
- No production backend in current runtime path.
- Persistence is browser `localStorage` key `assets` plus `theme`.
- Market/currency/commodity quote sources are repository-driven front-end data flows.

### Current strengths
- Clear route/module split (`home`, `assets`, `symbols`).
- Working add/edit/delete/search/theme user journeys.
- E2E regression safety net via Cypress suite.

### Current constraints and debt
- Single-device, single-browser persistence.
- Effects are manual subscriptions in singleton services.
- Add flow has double mutation risk (`reduceAddAsset` + dispatched effect).
- Update/delete uses `symbol` as identity in some paths while model has numeric `id`.

## 3) Target Architecture

### Target direction
- Keep existing route and component contracts stable.
- Replace repository internals with API-backed repositories.
- Introduce auth context and user-owned assets.
- Keep catalog lookups (categories/symbols) via repository abstraction.

### Target principles
- Brownfield first: no disruptive UI rewrite.
- Contract first: map existing domain types to API DTOs.
- Progressive migration: dual-read/write bridge when needed.
- Test-first for critical asset lifecycle flows.

## 4) C4-lite View

### 4.1 Context
System: AssetsBoard Portfolio Tracker.

Actors and external systems:
- Private Investor uses web browser.
- Future Auth Provider issues identity context/JWT.
- Future Assets API persists assets and serves catalogs.

Relationship summary:
- Investor interacts with Angular SPA.
- SPA currently reads/writes browser storage.
- Target SPA reads/writes backend API with user scoping.

### 4.2 Containers
Current containers:
- Container A: Angular SPA (`assets-board/`).
- Container B: Browser localStorage (`assets`, `theme`).
- Container C: Cypress E2E project (`tests/`).

Target containers:
- Container A: Angular SPA (unchanged route UX, API repositories).
- Container B: Backend API service (assets, categories, symbols endpoints).
- Container C: Relational DB (PostgreSQL/SQLite-compatible schema).
- Container D: Auth service / token issuer.
- Container E: Cypress E2E and Angular unit tests in CI.

### 4.3 Components (frontend)
- Routing shell: lazy-loaded modules from `app-routing.module.ts`.
- Core layout: header and theme toggle.
- Asset write models: new/edit services and forms.
- State store: `AssetsStoreService` (BehaviorSubject + action stream).
- Side effects: `AssetsEffects` dispatch handling and repository IO.
- Data repositories: assets, categories, symbols, commodities, currencies, stocks.

### 4.4 Key interfaces
- Store action contract: `Action` in `assets-actions.type.ts`.
- Domain object contract: `Asset` + supporting category/symbol/value types.
- Future API contract (target):
  - `GET /v1/assets`
  - `POST /v1/assets`
  - `PUT /v1/assets/{id}`
  - `DELETE /v1/assets/{id}`
  - `GET /v1/catalog/categories`
  - `GET /v1/catalog/symbols?search=&categoryId=`

## 5) Key Flows

### 5.1 Current: Add Asset
1. User submits new asset form.
2. `NewAssetService.saveAsset` currently performs optimistic local reducer update.
3. Service dispatches `ADD_ASSET` action.
4. `AssetsEffects` calls repository `post$` and then reducer add.
5. Repository writes updated list to localStorage.

Architecture note:
- Current sequence risks duplicate insertion during refactors due to two add reducers.

### 5.2 Current: Edit/Delete Asset
1. Route param `symbol` loads selected asset from store.
2. Update dispatches `UPDATE_ASSET` and repository persists localStorage.
3. Delete dispatches `DELETE_ASSET`, repository removes by symbol, then navigate home.

### 5.3 Current: Symbol Search
1. Symbols page reads query param `search`.
2. BehaviorSubject triggers filtered repository query.
3. Router query params are synchronized from search term.

### 5.4 Target: Authenticated Asset CRUD
1. SPA boots and resolves auth context.
2. Asset repositories attach bearer token and call API.
3. API enforces `owner_user_id` on all CRUD.
4. DB transactions persist asset data and audit timestamps.
5. SPA store reducers update from API responses only (single source of truth).

## 6) ADRs

### ADR-001: Preserve modular Angular route boundaries
Decision:
- Keep existing lazy modules (`home`, `assets`, `symbols`) and shared services.

Rationale:
- Minimizes regression risk and training overhead in brownfield codebase.

Tradeoffs:
- Some existing boundaries are service-heavy and not fully domain-centric.

### ADR-002: Keep repository abstraction and swap implementations
Decision:
- Maintain repository interfaces and move from localStorage to API implementations.

Rationale:
- Enables incremental migration and stable component contracts.

Tradeoffs:
- Temporary duplication (local and remote repository implementations).

### ADR-003: Canonical asset identity is numeric `id` (not symbol)
Decision:
- Target API and DB use immutable numeric/UUID PK as primary identity.

Rationale:
- Symbols can repeat across categories/providers and may change.

Tradeoffs:
- Requires migration logic from symbol-oriented update/delete paths.

### ADR-004: Reducers mutate state once per command
Decision:
- Remove double-mutation pattern in add flow; reducers run from effect result only.

Rationale:
- Prevents duplicate records and keeps event sequencing deterministic.

Tradeoffs:
- Slightly less immediate optimistic UX unless explicit optimistic strategy is added.

### ADR-005: Keep Cypress as E2E default for this repository phase
Decision:
- Continue Cypress for current acceptance/regression suite.

Rationale:
- Existing tests and scripts are stable and aligned with current workflows.

Tradeoffs:
- Team may revisit Playwright later if cross-browser/parallelization needs increase.

## 7) Risks and Mitigations

1. Risk: Duplicate add mutations create inconsistent totals.
   Mitigation: enforce single write path (effect-driven reducer), add unit test guard.

2. Risk: Data loss from localStorage clear.
   Mitigation: add migration/export/import strategy and backend persistence rollout.

3. Risk: Manual subscriptions can leak lifecycle complexity.
   Mitigation: centralize subscriptions or move to signal/store pattern progressively.

4. Risk: Documentation drift between local and target models.
   Mitigation: keep PRD + ADD + ERD updated together per release checklist.

5. Risk: Identity mismatch (`symbol` vs `id`) across flows.
   Mitigation: migrate component/service APIs to id-based CRUD in target phase.

## 8) Observability, Testing, and Deployment

### Observability
- Current: console-level logs only; no centralized telemetry.
- Near-term target:
  - Add client error boundary logging and HTTP interceptor for request metrics.
  - Add correlation id propagation from frontend to API.
  - Define minimal event taxonomy: `asset_created`, `asset_updated`, `asset_deleted`.

### Testing
- Unit: Angular Karma/Jasmine (`npm test` in `assets-board/`).
- E2E: Cypress (`npm test` in `tests/`) with base URL `http://localhost:4200/`.
- Recommended additions:
  - Reducer/effects tests for single-mutation add behavior.
  - API contract tests once backend endpoints exist.

### Deployment notes (repo-accurate)
- Frontend build artifact from `assets-board/` via `npm run build`.
- E2E project runs separately from `tests/`; app server must be running first.
- CI shape:
  1. install frontend deps
  2. lint/test/build frontend
  3. start app server
  4. run Cypress tests

## 9) File-Path Mapping (Practical)

### Presentation and routing
- `assets-board/src/app/app-routing.module.ts`
- `assets-board/src/app/routes/home/**`
- `assets-board/src/app/routes/assets/new/**`
- `assets-board/src/app/routes/assets/edit/**`
- `assets-board/src/app/routes/symbols/**`

### Core shell
- `assets-board/src/app/core/core.module.ts`
- `assets-board/src/app/core/layout/header/**`
- `assets-board/src/app/core/layout/theme-toggle/**`

### Domain model
- `assets-board/src/app/domain/asset.type.ts`
- `assets-board/src/app/domain/category.type.ts`
- `assets-board/src/app/domain/category-symbol-vo.type.ts`
- `assets-board/src/app/domain/quote.type.ts`
- `assets-board/src/app/domain/currency.type.ts`
- `assets-board/src/app/domain/commodity.type.ts`

### State/effects/repositories
- `assets-board/src/app/shared/assets/assets-store.service.ts`
- `assets-board/src/app/shared/assets/asset-effects.service.ts`
- `assets-board/src/app/shared/assets/assets-repository.service.ts`
- `assets-board/src/app/shared/categories-repository.service.ts`
- `assets-board/src/app/shared/symbols-repository.service.ts`

### Test and quality
- `tests/cypress/e2e/*.cy.js`
- `tests/cypress.config.js`
- `assets-board/src/**/*.spec.ts`

### Product and architecture docs
- `docs/PRD.md`
- `docs/ADD.md`
- `docs/ERD.md`

## 10) Delivery Roadmap

1. Stabilize state mutation semantics (single write path, id-based commands).
2. Introduce API repository adapters behind existing services.
3. Add auth context and user ownership constraints.
4. Migrate localStorage assets to backend records with id remapping.
5. Expand observability and contract/integration tests.
