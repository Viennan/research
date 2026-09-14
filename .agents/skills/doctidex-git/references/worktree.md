# `worktree`

`worktree` creates and manages editable Git worktrees.

See [common.md](common.md) for shared interface and errors.

## Create

```bash
doctidex-git worktree create \
  (--install-id <INSTALL-ID> | --url <GIT-URL> \
    (--branch <BRANCH> | --tag <TAG> | --commit <HASH>)) \
  [--work-path <REPOSITORY-PATH>] \
  [--tree-name <TREE-NAME>]
```

`--install-id` and `--url` are mutually exclusive. URL sources require exactly one revision selector.

`--url` accepts a Git SSH URL. An `https://` URL is accepted for convenience and is converted to SSH form at
argument parsing.

Default paths use `/.doctidex-git/worktrees/<domain>/<repository-path>/<tree-name>`. Without `--tree-name`, a short random final directory name is generated. Explicit path or tree-name collisions are errors.

See [layout.md](layout.md) for the stable default Worktree directory organization and when to search it directly.

A Worktree may branch, modify, and commit from its recorded base commit. `worktree create --work-path` must not be inside `/.doctidex-git/imports/` or below an existing boundary point; that request fails even when the directory is absent. `worktree create --install-id` creates an independent Worktree directory.

Success:

```json
{
  "status": "ok",
  "message": {},
  "work-path": "/.doctidex-git/worktrees/<DOMAIN>/<REPOSITORY>/<TREE-NAME>"
}
```

## Query and remove

```bash
doctidex-git worktree query --work-path <REPOSITORY-PATH>
doctidex-git worktree remove --work-path <REPOSITORY-PATH> [--force]
```

`query` returns an `install-id` when the Worktree came from an Installation; URL-created Worktrees omit it.

`remove` deletes the managed directory, Worktree record, and custom ignore rule. A dirty or abnormal worktree requires `--force`.

## Handleable errors

| Code | Cause and next step |
|---|---|
| `revision.unresolvable` | URL selector cannot resolve. |
| `worktree.source.unavailable` | Source cannot provide the target commit. |
| `worktree.target.unavailable` | Work path exists, is under `/.doctidex-git/imports/`, is below an existing boundary point, is a managed Worktree, or cannot be created. |
| `worktree.ignore.protection.failed` | Custom ignore rule could not be maintained. |
| `worktree.not-found` | Query path has no Worktree record. |
| `worktree.remove.blocked` | Worktree has uncommitted changes or abnormal state; use `--force`. |
| `worktree.remove.unavailable` | Managed directory cannot be removed. |

## Installation context

All `worktree` commands are forbidden when the selected Git root is inside a managed Installation. A recorded Worktree is an ordinary repository path and is not Installation context.
