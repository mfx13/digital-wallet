# ADR-0005: Use Husky and lint-staged for Git hooks

- Status: accepted
- Date: 2026-11-16

## Context

Git hooks give fast local feedback before code reaches the remote. The project needs three hooks: `pre-commit` (lint and format only the staged files, ADR-0004), `commit-msg` (validate Conventional Commits) and `pre-push` (validate the branch name).

Hooks live in `.git/hooks`, which is not versioned, so a tool must keep the hook configuration in the repository and install it on every machine during `pnpm install`. Development happens on Windows (Git Bash) and Linux.

Hooks can be skipped with `--no-verify`, so they are a convenience, not a guarantee. CI remains the source of truth for every check.

## Options considered

1. **Husky + lint-staged**: the most widely used combination in the JavaScript ecosystem. Husky keeps each hook as a small shell script in `.husky/` and points Git to it with `core.hooksPath`; lint-staged runs commands only on staged files and safely handles partially staged files. Two dependencies and two places to configure; hooks are shell scripts that must stay portable between Windows (Git Bash) and Linux.
2. **Lefthook**: a single Go binary and a single `lefthook.yml` for all hooks, with built-in staged file filtering, glob filters and parallel execution. Less common in JavaScript projects and job postings; its npm package relies on an install script, which pnpm blocks by default (ADR-0002).

## Decision

We use **Husky with lint-staged**. It is the standard setup in JavaScript and TypeScript projects, so the knowledge transfers directly to most codebases, and lint-staged's handling of partially staged files is the most battle-tested, which protects commits made with `git add -p`.

## Consequences

- Each hook is a shell script in `.husky/` (`pre-commit`, `commit-msg`, `pre-push`), versioned in the repository.
- The root `package.json` has a `prepare` script that installs the hooks on `pnpm install`.
- lint-staged configuration maps file globs to commands (ESLint and Prettier) and runs only on staged files.
- Hook scripts must stay simple and POSIX-compatible to behave the same on Windows (Git Bash) and Linux.
- CI runs the same checks on every pull request, since hooks can be skipped.
- Implementation follows in a separate change, once ESLint and Prettier are configured.
