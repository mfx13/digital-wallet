# ADR-0002: Use pnpm as the package manager

- Status: accepted
- Date: 2026-11-16

## Context
The repository is a monorepo with multiple apps and shared packages (`apps/api`, `apps/web`, `packages/config`, `packages/contracts`). The package manager installs dependencies, locks exact versions, links workspace packages to each other and runs scripts locally, in CI and in Docker builds.

In a monorepo, a flat `node_modules` lets a package import a dependency it never declared (a phantom dependency) because another package installed it. The code works by accident and breaks when the other package removes that dependency. Preventing this is the main criterion for this project, which handles money and must be predictable.

Corepack is no longer bundled with Node.js since v25, so the package manager cannot rely on it being available in Node.js 26 (ADR-0001).

## Options considered
1. **npm**: bundled with Node.js, universal, native workspaces. Flat `node_modules` allows phantom dependencies; slowest install; limited monorepo features.
2. **pnpm**: strict `node_modules` where each package only resolves what it declares; content-addressable store with hard links (fast, low disk usage); `workspace:` protocol and `--filter` for monorepos; does not run dependency install scripts unless explicitly allowed. Not bundled with Node.js; some older tools assume a flat `node_modules`; Docker builds need pnpm-specific steps.
3. **Yarn Berry (v4)**: strict, Plug'n'Play removes `node_modules`, zero-installs. Plug'n'Play requires editor and tooling support (SDKs); lower adoption in new projects; confusing coexistence with frozen Yarn v1.
4. **Bun**: fastest installs, workspaces, install scripts blocked by default. It is a full runtime: using it only to install adds a second runtime to every environment, its monorepo tooling is less mature, and using it as the runtime would contradict ADR-0001.

## Decision
We use **pnpm**, mainly for dependency control: its strict `node_modules` layout turns phantom dependencies into immediate errors instead of latent bugs. Its workspace features (`workspace:` protocol, `--filter`) and safe defaults for install scripts also fit a financial system built as a monorepo.

## Consequences
- The root `package.json` declares the exact pnpm version in the `packageManager` field, the single source of truth for the pnpm version.
- Workspace packages are listed in `pnpm-workspace.yaml`, not in the `workspaces` field of `package.json`.
- Internal dependencies use the `workspace:` protocol, making them explicit.
- Every environment must install pnpm explicitly (developer machines, CI via `pnpm/action-setup`, Docker images), since Corepack is not bundled with Node.js 26.
- Dependencies that need install scripts must be explicitly allowed in the pnpm configuration.
- Tools that assume a flat `node_modules` may need extra configuration.
- Dockerfiles must follow pnpm patterns (for example `pnpm fetch` and `pnpm deploy`).
