# AGENTS.md

This file provides guidance on how to work with the n8n repository.

## Project Overview

n8n is a workflow automation platform written in TypeScript, using a monorepo
structure managed by pnpm workspaces. It consists of a Node.js backend, Vue.js
frontend, and extensible node-based workflow engine.

## General Guidelines

- Always use pnpm
- We use Linear as a ticket tracking system
- We use Posthog for feature flags
- When starting to work on a new ticket – create a new branch from fresh
  master with the name specified in Linear ticket
- When creating a new branch for a ticket in Linear - use the branch name
  suggested by linear
- Use mermaid diagrams in MD files when you need to visualise something

## Essential Commands

### Building
Use `pnpm build` to build all packages. ALWAYS redirect the output of the
build command to a file:

```bash
pnpm build > build.log 2>&1
```

You can inspect the last few lines of the build log file to check for errors:
```bash
tail -n 20 build.log
```

### Testing
- `pnpm test` - Run all tests
- `pnpm test:affected` - Runs tests based on what has changed since the last
  commit

Running a particular test file requires going to the directory of that test
and running: `pnpm test <test-file>`.

When changing directories, use `pushd` to navigate into the directory and
`popd` to return to the previous directory. When in doubt, use `pwd` to check
your current directory.

### Code Quality
- `pnpm lint` - Lint code
- `pnpm typecheck` - Run type checks

Always run lint and typecheck before committing code to ensure quality.
Execute these commands from within the specific package directory you're
working on (e.g., `cd packages/cli && pnpm lint`). Run the full repository
check only when preparing the final PR. When your changes affect type
definitions, interfaces in `@n8n/api-types`, or cross-package dependencies,
build the system before running lint and typecheck.

## Architecture Overview

**Monorepo Structure:** pnpm workspaces with Turbo build orchestration

### Package Structure

The monorepo is organized into these key packages:

- **`packages/@n8n/api-types`**: Shared TypeScript interfaces between frontend and backend
- **`packages/workflow`**: Core workflow interfaces and types
- **`packages/core`**: Workflow execution engine
- **`packages/cli`**: Express server, REST API, and CLI commands
- **`packages/editor-ui`**: Vue 3 frontend application
- **`packages/@n8n/i18n`**: Internationalization for UI text
- **`packages/nodes-base`**: Built-in nodes for integrations
- **`packages/@n8n/nodes-langchain`**: AI/LangChain nodes
- **`@n8n/design-system`**: Vue component library for UI consistency
- **`@n8n/config`**: Centralized configuration management

## Technology Stack

- **Frontend:** Vue 3 + TypeScript + Vite + Pinia + Storybook UI Library
- **Backend:** Node.js + TypeScript + Express + TypeORM
- **Testing:** Jest (unit) + Playwright (E2E)
- **Database:** TypeORM with SQLite/PostgreSQL/MySQL support
- **Code Quality:** Biome (for formatting) + ESLint + lefthook git hooks

### Key Architectural Patterns

1. **Dependency Injection**: Uses `@n8n/di` for IoC container
2. **Controller-Service-Repository**: Backend follows MVC-like pattern
3. **Event-Driven**: Internal event bus for decoupled communication
4. **Context-Based Execution**: Different contexts for different node types
5. **State Management**: Frontend uses Pinia stores
6. **Design System**: Reusable components and design tokens are centralized in
   `@n8n/design-system`, where all pure Vue components should be placed to
   ensure consistency and reusability

## Key Development Patterns

- Each package has isolated build configuration and can be developed independently
- Hot reload works across the full stack during development
- Node development uses dedicated `node-dev` CLI tool
- Workflow tests are JSON-based for integration testing
- AI features have dedicated development workflow (`pnpm dev:ai`)

### TypeScript Best Practices
- **NEVER use `any` type** - use proper types or `unknown`
- **Avoid type casting with `as`** - use type guards or type predicates instead
- **Define shared interfaces in `@n8n/api-types`** package for FE/BE communication

### Error Handling
- Don't use `ApplicationError` class in CLI and nodes for throwing errors,
  because it's deprecated. Use `UnexpectedError`, `OperationalError` or
  `UserError` instead.
- Import from appropriate error classes in each package

### Frontend Development
- **All UI text must use i18n** - add translations to `@n8n/i18n` package
- **Use CSS variables directly** - never hardcode spacing as px values
- **data-test-id must be a single value** (no spaces or multiple values)

When implementing CSS, refer to @packages/frontend/CLAUDE.md for guidelines on
CSS variables and styling conventions.

### Testing Guidelines
- **Always work from within the package directory** when running tests
- **Mock all external dependencies** in unit tests
- **Confirm test cases with user** before writing unit tests
- **Typecheck is critical before committing** - always run `pnpm typecheck`
- **When modifying pinia stores**, check for unused computed properties

What we use for testing and writing tests:
- For testing nodes and other backend components, we use Jest for unit tests. Examples can be found in `packages/nodes-base/nodes/**/*test*`.
- We use `nock` for server mocking
- For frontend we use `vitest`
- For E2E tests we use Playwright. Run with `pnpm --filter=n8n-playwright test:local`.
  See `packages/testing/playwright/README.md` for details.

### Common Development Tasks

When implementing features:
1. Define API types in `packages/@n8n/api-types`
2. Implement backend logic in `packages/cli` module, follow
   `@packages/cli/scripts/backend-module/backend-module-guide.md`
3. Add API endpoints via controllers
4. Update frontend in `packages/editor-ui` with i18n support
5. Write tests with proper mocks
6. Run `pnpm typecheck` to verify types

## Github Guidelines
- When creating a PR, use the conventions in
  `.github/pull_request_template.md` and
  `.github/pull_request_title_conventions.md`.
- Use `gh pr create --draft` to create draft PRs.
- Always reference the Linear ticket in the PR description,
  use `https://linear.app/n8n/issue/[TICKET-ID]`
- always link to the github issue if mentioned in the linear ticket.



## 🛡️ Strict Embedding Separation, Zero-Fallback Law & 24/7 Dual-GPU Invariant
- **Reference**: 

### 1. Das Absolute Fallback-Verbot (Zero-Fallback Law)
Unter keinen Umständen, zu keinem Zeitpunkt und aus keinem Grund darf ein Fallback zwischen verschiedenen Embedding-Modellen stattfinden.
* **Geltende Aktion:** Fällt ein Embedding-Modell aus oder ist überlastet, MUSS die Operation sofort hart fehlschlagen () oder die Payload transaktional in einer Queue (NATS/SQLite) verharren, bis das exakte Modell bereit ist.
* **Verboten:** Kein stiller oder dynamischer Modellwechsel (weder Jina -> Gemma noch umgekehrt).

### 2. Warum ein Embedding-Fallback mathematisch & informationstheoretisch unmöglich ist
* **Topologische Inkompatibilität heterogener Vektorräume (Non-Isomorphism):**
  Jedes Modell 	heta: \mathcal{X} 	o \mathbb{R}^D$ projiziert Text in eine spezifische, gelernte Riemannsche Mannigfaltigkeit. Jina v5 (=256$) und EmbeddingGemma (=768$) spannen zwei völlig inkompatible geometrische Räume auf. Die Basisvektoren der semantischen Achsen sind ohne explizite Procrustes-Transformation nicht ausgerichtet.
* **Kollaps der Kosinus-Ähnlichkeit ($	ext{sim} pprox 0$):**
  Wird eine Suchanfrage mit Modell $ berechnet ( = f_B(q)$), während der Dokumentenkorpus mit Modell $ indiziert wurde ( = f_A(d)$), verhält sich das Skalarprodukt mathematisch wie das zweier rein zufälliger Vektoren auf einer hochdimensionalen Einheitssphäre:
  86306\mathbb{E}[	ext{sim}(u, v)] = 0 \quad 	ext{mit Varianz} \quad \sigma^2 = rac{1}{D}86306
  Der Nearest-Neighbor-Algorithmus (HNSW/k-NN) liefert stochastisches Rauschen. Das RAG-System erhält völlig falsche oder irrelevante Kontexte.
* **Irreversible Index-Vergiftung (Index Poisoning):**
  Wird auch nur ein einziger Vektor von Modell $ als 'Fallback' in den Index von Modell $ geschrieben, verunreinigt er die Distanzgraphen und Clusterzentren dauerhaft.
* **Das Gesetz des Fail-Fast:**
  Ein Ausfall muss hart abbrechen ().

### 3. Duale 24/7 Erfassungspflicht (GPU-Only)
* **GPU-Only Mandat:** Es läuft absolut nichts auf der CPU — GPU ONLY (NVIDIA GB10 CUDA) für ausnahmslos jedes Embedding-Modell.
* **24/7 Parallelität:** Sowohl  (256D, ~4,1 GB VRAM) als auch  (768D, ~1,2 GB VRAM) laufen dauerhaft 24/7 im VRAM (Summe ~5,3 GB VRAM).
* **Duale Erfassung:** Jeder zu indizierende Text/Chunk wird immer von beiden Modellen parallel eingebettet und getrennt persistiert.

### 4. Idioten- & Failsafe-Sicherung auf Datenbankebene
* **SQLite Schema CHECK-Constraints:**
   und  erzwingen atomare Abbrüche auf Engine-Ebene bei Modell-Mismatches.
* **Qdrant Collection Constraints:**
  Strikte Trennung in separate Collections ( vs ) mit fixierter Vektordimension.


## ⚡ High-Quality Systems Programming Languages Priority (No-Python Policy)
- **Reference**: `/home/m1st/.agents/rules/RULE_High_Quality_Systems_Programming_Languages.md`
- **Rule**:
  1. **Bevorzugte Sprachen:** High Quality **Golang (Go), Rust, C++, Zig, PowerShell, C** sind IMMER und AUSNAHMSLOS die bevorzugten Programmiersprachen.
  2. **Kein Python:** Python ist für neue Daemons, Watcher, Automatisierungen, APIs, CLI-Tools und Dienste strikt untersagt (GIL-Bottlenecks, Dependency-Drift, Speicherineffizienz).
  3. **Natives Systems-Engineering:** Alle Hintergrunddienste, Caching-Ebenen und Task-Runner müssen als native, speichersichere und nebenläufige Binaries kompiliert werden.
