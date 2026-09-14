# `repair`

`repair` aligns recoverable physical state with the JSON work model.

See [common.md](common.md) for shared interface and errors.

## Usage

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] repair
```

Success:

```json
{"status": "ok", "message": {}}
```

Repeated repair should not add changes to an already consistent environment.

## Repair scope

| Object | Behavior |
|---|---|
| Residual RuntimeStore journals | Classify journal state, restore backups when required, and clean after physical repair. |
| Installation | Recreate missing tracked Installation directories where possible; discard dirty changes in existing Installations and restore them to their recorded commit. |
| Ref | Recreate missing symlinks; remove Refs whose Installation is gone. |
| Unregistered Installation links | Remove symlinks into Installations that have no Ref record. |
| Worktree | Recreate missing recorded Worktrees. |
| Boundaries and ignores | Reconcile derived boundaries and tool-managed ignore rules. |

## What repair does not do

`repair` does not edit Markdown link content, invent new domain records for unrecorded physical objects, roll back Git fetches, undo user commits, or discard dirty changes in a Worktree. A dirty Worktree still requires `--force` for removal.

Object-specific failures are surfaced with the owning command's structured error codes.

## Installation context

`repair` is forbidden when the selected Git root is inside a managed Installation.
