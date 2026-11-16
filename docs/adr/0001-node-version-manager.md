# ADR-0001: Use fnm as Node.js version manager and pin Node.js 26 LTS

- Status: accepted
- Date: 2026-11-16

## Context
The project must run on the same Node.js version on every developer machine, in CI (GitHub Actions) and in the deployment image. Development happens on Windows (Git Bash) and may also happen on Linux.

We need a tool that installs multiple Node.js versions side by side and switches automatically based on a version file committed to the repository. That file must be the single source of truth, also read by `actions/setup-node` in CI.

We also need to choose the Node.js major version. The project is long-lived and starts in November 2026, when v26, v24 and v22 are the supported LTS lines.

## Options considered

### Version manager
1. **fnm**: single-purpose, cross-platform (Windows, macOS, Linux), reads `.node-version` and `.nvmrc`, switches automatically on `cd` through a shell hook, ships as a single fast binary. Automatic switching only works when the shell hook is loaded; manages Node.js only.
2. **mise**: manages many runtimes, environment variables and tasks; actively maintained. Much broader scope than this project needs; Windows support has historically been less mature.
3. **nvm-windows**: widely known. It is a different project from POSIX nvm, so Windows and Linux would use different tools; no automatic switching; switches the version globally through a symlink, which may require admin rights.
4. **Volta**: pins the version in `package.json` without a shell hook. Officially unmaintained; its README recommends migrating to mise.

### Node.js version
1. **v26 LTS**: newest LTS line, longest support window. Some libraries may lag behind the newest major.
2. **v24 LTS**: more battle-tested ecosystem. Shorter remaining support, forcing an earlier upgrade.
3. **v22 LTS**: close to end of life.

## Decision
We use **fnm** and pin **Node.js 26** in a `.node-version` file at the repository root.

fnm is the most current and focused option: it behaves the same on Windows and Linux and reads the same version file as CI. Node.js 26 is the newest LTS line, which fits a long-lived project and delays a forced major upgrade.

## Consequences
- `.node-version` is the single source of truth: fnm (local), `actions/setup-node` (`node-version-file`) and the Docker base image must all follow it.
- The root `package.json` will declare `engines.node` consistent with `.node-version`.
- Every developer must install fnm and enable its shell hook, documented in the README "Development setup" section.
- If a dependency does not support Node.js 26, we will need a workaround or a new ADR revisiting this decision.
- Upgrading Node.js means changing `.node-version` and `engines` together in a single PR.
