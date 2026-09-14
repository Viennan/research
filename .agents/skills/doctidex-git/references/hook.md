# `hook`

`hook` installs and runs Git hooks that keep the untracked runtime work model consistent across branch switches and
validate work-model structure before commits.

See [common.md](common.md) for shared interface and errors.

## Install

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] hook install
```

`hook install` injects a marked doctidex block into the supported `post-checkout` and `pre-commit` hook scripts. The
block is placed after leading comments and before the original first non-comment line, preserving existing hook
content. It uses the resolved `doctidex-git` command path, so hook execution does not depend on `PATH`. The injected
command runs first; if it fails, the script exits and the existing hook content does not run. The command is
idempotent.

Successful first-time `init` installs the same hooks automatically; explicit `hook install` remains available and safe
to rerun.

## Pre-commit

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] hook pre-commit
```

The installed `pre-commit` hook invokes this worker before each commit. The worker returns success when `.doctidex-git/`
is absent or when `validate --only-model-structure` reports a valid work model. When the work model is invalid, the worker
returns `hook.pre-commit.validation.failed` with the validation diagnostics and Git aborts the commit.

## Post-checkout

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] hook post-checkout [<HOOK-ARG>...]
```

The installed `post-checkout` hook invokes this worker with Git's old-head, new-head, and checkout-flag arguments.
The worker performs work only when both the old and new branches have a tracked `.doctidex-git/config.toml`.

If reconciliation fails, the checkout hook returns a nonzero status with a structured diagnostic. Correct the work
model or physical state, then rerun:

```bash
doctidex-git --repos-path <REPOSITORY-ROOT-PATH> hook post-checkout
```

The no-argument rerun is apply-only for the current branch.

## Handleable errors

| Code | Cause and next step |
|---|---|
| `hook.command.unavailable` | `hook install` could not resolve the `doctidex-git` command path. |
| `hook.install.unavailable` | The Git hooks directory or hook script could not be written. |
| `hook.pre-commit.validation.failed` | The work model failed `validate --only-model-structure`; correct it and commit again. |
| `hook.post-checkout.reconcile.failed` | Runtime state or physical Installation/share paths could not be reconciled; correct the model and rerun `hook post-checkout`. |

## Installation context

`hook install`, `hook pre-commit`, and `hook post-checkout` are forbidden inside a managed Installation.
