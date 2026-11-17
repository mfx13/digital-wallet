# ADR-0004: Use ESLint with typescript-eslint and Prettier

- Status: accepted
- Date: 2026-11-16

## Context

The project needs a linter and a formatter shared by every workspace package (ADR-0003), run in the pre-commit hook and in CI.

The core of the wallet (deposits, transfers, balance updates) is asynchronous code that touches money. A missing `await` on a debit, or a promise passed where a boolean is expected, compiles without errors but can lose or duplicate money. Catching these bugs requires type-aware lint rules, which use type information from the TypeScript compiler. This is the main criterion for this decision.

## Options considered

1. **ESLint + typescript-eslint + Prettier**: the most complete set of type-aware rules (`no-floating-promises`, `no-misused-promises`, `switch-exhaustiveness-check`, `strict-boolean-expressions`); mature plugin ecosystem; industry standard. Two tools and more configuration; type-aware linting is slow because it builds the whole TypeScript program.
2. **Biome**: a single fast tool (written in Rust) for linting and formatting, with minimal configuration and Prettier-compatible output. Its type inference covers fewer type-aware rules than typescript-eslint; smaller plugin ecosystem.
3. **Biome for formatting + ESLint for type-aware rules**: fast formatting and the critical rules covered. Still two tools to maintain, with overlapping lint rules to coordinate.

## Decision

We use **ESLint with typescript-eslint (`strictTypeChecked` preset) and Prettier**. Type-aware rules for asynchronous code protect the core of the wallet, where a missed promise means an inconsistent balance. The cost (more configuration and slower linting) is acceptable with four packages, and the pre-commit hook can lint only the changed files.

## Consequences

- Lint and format configuration lives in a shared workspace package and is reused by every app and package.
- ESLint must not enforce formatting rules; formatting belongs to Prettier only (for example, via `eslint-config-prettier`).
- Type-aware linting needs each package's `tsconfig.json`, so lint and TypeScript configuration must stay in sync.
- Full lint runs are slower; the pre-commit hook lints only staged files and CI runs the full lint.
- Implementation follows in a separate change, once the shared configuration package and the base `tsconfig` exist.
- Revisit Biome in a new ADR if its type-aware rule coverage reaches what the project relies on, or if lint time becomes a bottleneck.
