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

## Hosted result (November 3, 2026)

Dependabot [PR #1](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/pull/1), commit `80124573a9eefcccfe2c3087e8faffc6a1b50178`, changes only the workspace manifest from lodash 4.17.20 to 4.18.1. Its [PR CI run fails](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/actions/runs/41655381509/job/125984158493):

```text
$ pnpm install --frozen-lockfile --ignore-scripts
Scope: all 2 workspace projects
Error: ERR_PNPM_OUTDATED_LOCKFILE

  × installing dependencies
  ╰─▶ Cannot install with "frozen-lockfile" because pnpm-lock.yaml is not up
      to date with package.json.
      
        Failure reason:
        specifiers in the lockfile don't match specifiers in package.json:
      * in importers["packages/app"]:
      * 1 dependency is mismatched:
        - lodash (lockfile: 4.17.20, manifest: 4.18.1)
  help: Regenerate the lockfile with `pnpm install --lockfile-only` so that
        pnpm-lock.yaml reflects the current package.json, then re-run `pnpm
        install --frozen-lockfile`.

Process completed with exit code 1.
```

The [base CI run passes](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/actions/runs/41655004351). The separate root-scoped [PR #2](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/pull/2), commit `c979fafe2e2aba09a3b4a2c5c58352a853fb614d`, updates both files and [passes CI](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/actions/runs/41655404085/job/125984236735). This reproduction does not reproduce a missing root PR.

The [workspace updater job](https://github.com/karlhorky/repro-dependabot-pnpm12-workspace-security-updates-missed-lockfile-update/actions/runs/41655198207) reports `security-updates-only: true`, `source.directories: ["/packages/app"]`, installed pnpm 12.10.1, and updater image `473accfb9bdf65bd91ef23fae3d9b57a02e27557`. CI uses Node.js 24 and pnpm 12.11.2.

Dependabot is configured only for `/`, but still creates the broken workspace-scoped PR. Expected: every security PR updates the member manifest and shared root lockfile together.

The [dependency-graph parsing fix announced today](https://github.com/dependabot/dependabot-core/issues/15904#issuecomment-6605704055) is separate from this updater failure. Both manifest and root-lockfile alerts were detected in this fresh repository.

Related updater issue: https://github.com/dependabot/dependabot-core/issues/16680.
