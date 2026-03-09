# feat-backend-repository-adapters

## Status
- Spec Status: Draft
- Feature Status: InProgress
- Owner: Product/Engineering
- Last Updated: 2026-03-09

## Problem
AssetsBoard currently persists assets and catalogs through frontend/localStorage repositories. This blocks backend evolution because repository implementations are not yet wired as backend-ready adapters with clear switching boundaries.

Without adapter wiring:
- Frontend state and effects remain tightly coupled to local persistence behavior.
- Backend CRUD and authenticated ownership (targeted in PRD/ADD/ERD) cannot be introduced incrementally.
- Migration risk increases because UI flows may need disruptive rewrites instead of implementation swaps.

## Scope
This feature introduces backend-ready repository adapters while preserving current UI and route contracts.

In scope:
- Define repository adapter interfaces/contracts for assets, categories, and symbols access patterns already used by app services/effects.
- Add adapter implementations split by transport/persistence strategy:
  - local adapter (current localStorage/static data behavior)
  - backend adapter (HTTP-ready shape, can use stub endpoints until backend exists)
- Add adapter wiring/composition so current consumers resolve repositories through abstraction, not direct persistence assumptions.
- Preserve action, service, and component call contracts used by existing screens (`home`, `assets/new`, `assets/edit`, `symbols`).
- Keep identity transition readiness by supporting id-first CRUD signatures in adapter contracts where required by target model.

## Non-Scope
- No UI redesign or route changes.
- No authentication implementation.
- No live backend delivery or endpoint implementation.
- No data migration execution from localStorage to backend.
- No broad state-management refactor beyond minimal adapter integration touchpoints.

## User Impact
Expected user-visible behavior remains unchanged:
- Users still add/edit/delete assets with current UX.
- Users still search symbols and toggle theme as before.
- No additional user steps or settings are introduced.

Primary impact is technical: the app becomes ready to switch repository internals to backend without breaking current UI contracts.

## Solution Outline
1. Repository Contract Layer
- Introduce explicit adapter contracts for:
  - assets CRUD and listing
  - categories lookup
  - symbols lookup/search
- Keep method semantics aligned with existing store/effects flow.

2. Adapter Implementations
- Local adapters wrap current localStorage/static-repository behavior.
- Backend adapters expose the same contract and map to target API DTO boundaries defined in ADD/ERD direction.

3. Adapter Wiring
- Wire adapters through Angular DI/factory/provider strategy.
- Default wiring uses local adapters to keep current runtime behavior stable.
- Add environment-level toggle (or equivalent configuration seam) to allow backend adapter activation without component/service contract changes.

4. Compatibility Guardrails
- Preserve existing UI contracts and route flow expectations.
- Keep reducer/effects interaction deterministic (single write path expectations remain valid).
- Ensure adapter outputs preserve domain type compatibility used by current presentation/state layers.

## Affected Files/Modules
Likely affected modules/files (implementation-oriented map):
- `assets-board/src/app/shared/assets/` (repository + effects/store integration)
- `assets-board/src/app/shared/` (categories/symbols repository abstractions)
- `assets-board/src/app/domain/` (contract/DTO compatibility types as needed)
- Angular DI/provider setup in app/core/shared modules where repositories are instantiated
- Configuration/environment wiring used to select adapter strategy
- Unit tests for repository contracts and adapter wiring
- Cypress suites only if behavior regressions require expectation updates (not expected)

## Acceptance Criteria (Testable)
1. Given the app runs with default configuration, when user executes add/edit/delete asset flows, then behavior and visible outcomes match current baseline UX and persisted results.
2. Given repository consumers in state/effects/services, when persistence is invoked, then calls go through repository adapter contracts rather than direct localStorage/static transport details.
3. Given local adapter mode is active, when app starts and performs CRUD/search flows, then no regressions occur in existing unit and Cypress core journeys.
4. Given backend adapter mode is enabled via configuration seam, when repository methods are called, then the app resolves backend adapter implementations without changing component/service call sites.
5. Given adapter contracts for assets, when update/delete operations execute, then operations support stable id-based identity semantics required by target backend model.
6. Given symbols/category retrieval, when using either adapter mode, then returned domain-compatible data structures remain valid for existing UI renderers.
7. Given repository adapter wiring, when one adapter implementation is replaced, then no changes are required in route components (`home`, `assets/new`, `assets/edit`, `symbols`).
8. Given build and test pipelines, when this feature is integrated, then frontend unit tests pass and Cypress smoke/new/edit/symbols/theme flows remain green.

## Risks
- Contract drift between frontend domain types and eventual backend DTOs could cause future adapter churn.
- Hidden coupling to localStorage behavior may surface during adapter substitution.
- Identity mismatch (`symbol`-centric paths vs id-based target model) can cause partial compatibility if not normalized at adapter boundary.
- Duplicate state mutation risk in add flow may be amplified if adapter semantics are inconsistent.

## Rollout Plan
1. Phase 1: Introduce contracts and local adapter wrappers, keep behavior identical.
2. Phase 2: Add backend adapter implementations and DI selection seam behind feature/config toggle.
3. Phase 3: Validate with unit tests and existing Cypress critical paths in local mode.
4. Phase 4: Enable backend adapter in controlled environments with mocked/stubbed API responses.
5. Phase 5: Promote backend adapter as default only after backend readiness, contract validation, and migration strategy approval.

## Implementation Tracking Checklist
- [ ] Define adapter contracts for assets/categories/symbols.
- [ ] Implement local adapters conforming to new contracts.
- [ ] Implement backend-ready adapters conforming to same contracts.
- [ ] Add DI/provider wiring and adapter selection configuration seam.
- [ ] Ensure consumers depend on contracts, not transport details.
- [ ] Validate id-based identity support in assets adapter methods.
- [ ] Add/adjust unit tests for adapter contract and provider wiring.
- [ ] Run Cypress core journeys and confirm no UI regressions.
- [ ] Document adapter toggle and operational defaults.
