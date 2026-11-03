# Dependabot pnpm 12 workspace security update reproduction

One workspace package pins `lodash@4.17.20`. pnpm 12.11.2 writes a two-document root `pnpm-lock.yaml`. Dependabot targets `/`, with scheduled version PRs disabled.

## Reproduce

1. Create a public GitHub repository from these files
2. Enable the dependency graph, Dependabot alerts, and Dependabot security updates
3. Wait for the workspace lodash alert and security PR
4. Inspect the PR diff and the `Frozen install` GitHub Actions check

Expected: Dependabot updates `packages/app/package.json` and the root `pnpm-lock.yaml` together, and CI passes.

CI uses Node.js 24 and pnpm 12.11.2:

```sh
pnpm install --frozen-lockfile --ignore-scripts
```
