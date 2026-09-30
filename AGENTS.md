# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository shape

This is a small sandbox repository with no application code yet. It contains
only tooling and configuration:

| Path                         | Purpose                                      |
| ---------------------------- | -------------------------------------------- |
| `README.md`                  | Human-facing overview and setup steps        |
| `AGENTS.md`                  | This file: guidance for agents               |
| `mise.toml`                  | Pinned tool versions (managed by mise)       |
| `prek.toml`                  | Pre-commit hook configuration (run by prek)  |
| `.github/workflows/prek.yml` | CI workflow that runs all prek hooks         |

## Tooling

- [mise](https://mise.jdx.dev) pins tool versions in `mise.toml`. Currently
  it pins only `prek`.
- [prek](https://github.com/j178/prek) runs the pre-commit hooks defined in
  `prek.toml`. It is a drop-in, faster reimplementation of `pre-commit`.
- Node.js is required for the `local` hooks (`taplo` and `markdownlint`),
  which prek installs automatically as npm packages.

## Checks

All checks are prek hooks, defined in `prek.toml`:

| Hook                  | What it does                                      |
| --------------------- | ------------------------------------------------- |
| `trailing-whitespace` | Strips trailing whitespace                        |
| `end-of-file-fixer`   | Ensures files end with exactly one newline        |
| `mixed-line-ending`   | Normalises line endings to LF                     |
| `check-toml`          | Validates TOML syntax                             |
| `taplo-fmt`           | Formats TOML files with taplo                     |
| `markdownlint`        | Lints and auto-fixes Markdown (markdownlint-cli2) |

Several hooks modify files in place. If a hook fails because it rewrote a
file, re-stage the file and run the hooks again.

Markdownlint uses its default rules, so keep Markdown lines to 80 characters
(including tables and code blocks) and use consistent heading and list
styles.

CI (`.github/workflows/prek.yml`) runs every hook against all files on pushes
to `main` and on pull requests. There is no test suite; passing hooks is the
bar for a change.

## Commands to run

Set up the pinned tools:

```sh
mise install
```

Before committing, run all checks against the whole tree (this mirrors CI):

```sh
prek run --all-files --show-diff-on-failure
```

Optionally install the git hook so checks run on every commit:

```sh
prek install
```

If `mise` or `prek` is unavailable, the Markdown and TOML checks can be run
directly with Node.js:

```sh
npx --yes markdownlint-cli2@0.23.3 "**/*.md"
npx --yes @taplo/cli@0.7.0 fmt --check
```

## Conventions

- When adding a new tool, pin its version in `mise.toml`.
- When adding a new check, add it as a hook in `prek.toml` so it runs locally
  and in CI.
- Keep `README.md` and this file up to date when tooling or checks change.
