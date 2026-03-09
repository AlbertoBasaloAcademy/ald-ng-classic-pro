---
title: Legacy Angular AI Workshop - Architecture Diagram
purpose: Provide a concise, explorable architecture diagram of the current Angular app, plus a reusable prompt to regenerate/update the diagram as the code evolves.
prompt_template: |
  Generate a concise architecture diagram for this Angular project.
  Include:
  1) bootstrap and root module,
  2) lazy routes and feature modules,
  3) state/effects/repository flow,
  4) external data providers,
  5) local persistence and tests.
  Return a Mermaid diagram and a short explanation of each layer.
target_audience: Workshop participants learning architecture mapping with AI in legacy Angular projects.
model_hint: GPT-5.3-Codex
last_updated: 2026-03-09
---

# Quick Architecture Diagram

```mermaid
flowchart TD
  A[main.ts] --> B[AppModule]
  B --> C[CoreModule]
  C --> D[Header + Theme Toggle]
  B --> E[AppRouting]

  E --> H[HomeModule /]
  E --> I[AssetsModule /assets]
  E --> J[SymbolsModule /symbols]

  H --> H1[HomeComponent]
  H1 --> H2[HomeService]

  I --> I1[NewModule /new]
  I --> I2[EditModule /edit/:symbol]
  I1 --> I3[NewAssetService]
  I2 --> I4[EditAssetService]

  J --> J1[SymbolsComponent]
  J1 --> J2[SymbolsRepository]

  H2 --> S[AssetsStoreService]
  I3 --> S
  I4 --> S
  S --> X[Action Stream]
  X --> Y[AssetsEffects]
  Y --> Z[AssetsRepository]

  Z --> L[(localStorage assets)]
  Z --> V[AssetValueService]

  V --> R1[CurrenciesRepository]
  V --> R2[CommoditiesRepository]
  V --> R3[StocksRepository]

  J2 --> R1
  J2 --> R2
  J2 --> R3

  T[ThemeToggle] --> LT[(localStorage theme)]

  U[Unit Tests: Karma/Jasmine]:::test
  W[E2E Tests: Cypress]:::test

  classDef test fill:#eef,stroke:#99f,color:#223;
```

## Layer Notes

1. Shell and routing: app starts in main, loads root module, then lazy-loads route modules.
2. Feature modules: home, assets (new/edit), and symbols each own their UI + orchestration service.
3. State model: custom store + action stream + effects, instead of NgRx.
4. Persistence: assets and theme are persisted in localStorage.
5. Data providers: repositories are in-memory/fake and return Observables for a backend-like flow.

## Workshop Discussion Points

1. Why this custom store is easy to understand but risky at scale.
2. How to migrate incrementally toward cleaner effect orchestration.
3. Where to add anti-corruption layers before introducing real APIs.
4. How to keep tests resilient while refactoring legacy structure.
