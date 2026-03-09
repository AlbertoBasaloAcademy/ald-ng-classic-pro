# AGENTS Guide for AssetsBoard

You are a coding agent configured at `/.agents` folder.
Check the `manifest.md` for the list of available sub-agents, prompts and skills. 

## Product Overview
- AssetsBoard is a local-first portfolio tracker for private investors.
- Main flows: add, edit, delete assets; symbols search; theme toggle.
- Source of truth for requirements: `docs/PRD.md`.

## Tech Snapshot
- Frontend: Angular 15 in `assets-board/`.
- Language: TypeScript 4.9 + RxJS 7.
- Current persistence: browser localStorage (`assets`, `theme`).
- E2E tests: Cypress project in `tests/`.
- Unit tests: Karma/Jasmine in `assets-board/`.

## Environment
- OS: Windows.
- Use workspace root: `c:/code/live/ald-ng-classic-pro`.
- App default URL: `http://localhost:4200/`.
- Cypress baseUrl: `http://localhost:4200/`.

## Workflow Commands

### Frontend app (`assets-board/`)
```bash
cd assets-board
npm install
npm run start
npm run build
npm run test
```

### Cypress tests (`tests/`)
```bash
cd tests
npm install
npm run start   # interactive
npm run test    # headless
```

### Typical local run sequence
```bash
cd assets-board && npm run start
# in a second terminal
cd tests && npm run test
```

## Architecture Guardrails
- Keep brownfield module boundaries (`home`, `assets`, `symbols`).
- Preserve repository abstraction; do not bind components to transport details.
- Keep store updates deterministic and avoid duplicate reducer mutations.
- Prefer id-based asset identity for new backend-facing logic.
- Align data model changes with both `docs/ADD.md` and `docs/ERD.md`.

## Agent Behavior Guidelines
- Read `docs/PRD.md` before architecture or feature changes.
- Match existing coding style and avoid unrelated refactors.
- Keep docs concise, concrete, and synchronized with real scripts.
- When changing persistence logic, update tests that cover CRUD flows.
- If assumptions are required, state them explicitly in output.
- Never use destructive git commands without explicit user approval.

## File Map for Fast Navigation
- Frontend app: `assets-board/src/app/`
- Domain types: `assets-board/src/app/domain/`
- Assets state/repo: `assets-board/src/app/shared/assets/`
- Feature routes: `assets-board/src/app/routes/`
- E2E tests: `tests/cypress/e2e/`
- Architecture docs: `docs/`
