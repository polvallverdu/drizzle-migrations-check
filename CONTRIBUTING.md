# Contributing

Thanks for helping! Bug reports, new scenarios, and fixes for repo setups that aren't handled
yet are all welcome.

## How the project is laid out

| Path | What it is |
| --- | --- |
| `action.yml` | Action metadata: inputs, outputs, and a single composite step that runs `check.sh`. |
| `check.sh` | All the logic. Plain bash, configured through env vars, so it runs fine outside GitHub Actions. |
| `test/run.sh` | Scenario tests. Each `test_*` function builds a throwaway git repo from a fixture and runs `check.sh` against it. |
| `test/fixtures/` | `single-repo` (one package, one config) and `monorepo` (pnpm workspace: a package with 3 configs plus a package with 1). |
| `test/lib.sh` | Helpers shared by the tests and the e2e workflow (`new_repo`, `generate`, `drift`, …). |
| `.github/workflows/test.yml` | Runs the scenarios on Linux and macOS, plus the real action end to end with npm and pnpm installs. |
| `skills/` | Agent skill that teaches coding agents how to set the action up. |

## Running the tests

```bash
cd test
npm install
npm test             # every scenario
bash run.sh glob     # only scenarios whose name contains "glob"
```

The tests use the drizzle-kit installed in `test/node_modules`. Each scenario writes to
`test/.work/<scenario>/`, which is gitignored and handy to inspect when something fails.

`check.sh` has to run on **bash 3.2** (the default on macOS) as well as bash 5. In practice:
use `"${arr[@]+"${arr[@]}"}"` to expand arrays that might be empty, and don't use `mapfile`,
associative arrays or `${var,,}`.

## Making a change

1. Fork the repo and create a branch from `main`.
2. If you're fixing a bug, first add a scenario to `test/run.sh` that reproduces it and fails.
3. Make the change in `check.sh` and/or `action.yml`.
4. If inputs, outputs or behavior changed, update `README.md`, `action.yml` and
   `skills/setup-drizzle-migrations-check/SKILL.md`.
5. Open a pull request. CI has to pass before it can be merged, and PRs are squash-merged.

Keep the action a **composite action with no build step**. Users shouldn't have to trust
compiled JavaScript to read what the action does.

## Releasing (maintainers)

Versions follow [semver](https://semver.org). Users pin to the major tag (`@v1`), so anything
that can make a passing setup fail without a code change on their side is a breaking change:
renaming inputs, changing defaults, or new failure conditions that aren't clearly bugs.

1. Make sure `main` is green.
2. On GitHub, go to **Releases → Draft a new release**, create a new tag `vX.Y.Z` on `main`, and
   click **Generate release notes**.
3. Keep **Publish this Action to the GitHub Marketplace** ticked, and publish.
4. The `Release` workflow then moves the major tag (`v1`) to the new release automatically.

Exact version tags (`vX.Y.Z`) can't be moved or deleted (a tag ruleset plus immutable releases),
so `@v1.0.0` always points to the same code. Only the major tag (`v1`) moves.

## Code of conduct

By taking part you agree to the [code of conduct](CODE_OF_CONDUCT.md).
