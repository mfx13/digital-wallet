# ADR-0003: Use pnpm workspaces without a monorepo orchestrator

- Status: accepted
- Date: 2026-11-16

## Context

The monorepo has two apps and two shared packages (`apps/api`, `apps/web`, `packages/config`, `packages/contracts`), managed by pnpm workspaces (ADR-0002). Root scripts such as `lint`, `typecheck` and `test` must run across all packages, in dependency order, locally and in CI.

Monorepo orchestrators add task pipelines, caching of task results and detection of affected packages. These features pay off when there are many packages and slow tasks. Here, the expected bottlenecks are the database and concurrency, not build or CI time.

## Options considered

1. **pnpm workspaces only**: no extra tool or configuration; `pnpm -r` runs scripts in every package in topological order; `--filter` selects packages, including those changed since a Git ref. No task caching, so every run executes every task.
2. **Turborepo**: small configuration (`turbo.json`) describing task dependencies and cache inputs; local cache and optional remote cache shared with CI; common in TypeScript job postings. One more tool and config file; little gain at this size.
3. **Nx**: project graph, affected detection, code generators and framework plugins. Steepest learning curve and more opinionated about project structure; oversized for four packages.

## Decision

We use **pnpm workspaces only**. The problems orchestrators solve (task caching and long pipelines) do not exist yet, and pnpm already covers running scripts in dependency order and filtering packages. Keeping one less tool makes each layer of the toolchain explicit.

## Consequences

- Root scripts run across packages with `pnpm -r run <script>` (or `pnpm --filter` for a subset).
- Every run executes all tasks; there is no cache of task results.
- CI can limit work to changed packages with `pnpm --filter "...[origin/main]"` if needed.
- Adopting Turborepo later only requires adding `turbo.json` and changing root scripts; the folder structure stays the same.
- Revisit this decision in a new ADR if CI or local task times become a bottleneck (for example, once `apps/web` and `packages/contracts` have build steps).
