# `cache`

`cache` maintains the user-level Git cache. See [common.md](common.md#cache-configuration) for cache location and
configuration.

## Storage layout

The cache root is selected by `cache-path`. `cache-status.json` sits directly in that root, and bare repositories are
stored under `data/<domain>/<repository...>`.

## Clean

```bash
doctidex-git cache clean
```

`cache clean` removes published cached bare repositories that have no non-bare linked Worktree. It first runs
`git worktree prune`, then reads `git worktree list --porcelain`; a repository whose only listed entry is the bare
repository itself is unused.

`cache clean` does not change Installations, Refs, Worktrees, or RuntimeStore declarations.

Success:

```json
{"status": "ok", "message": {}, "removed": ["<GIT-URL>", "..."]}
```

## Compact

```bash
doctidex-git cache compact
```

`cache compact` runs `git worktree prune` and `git gc --prune=now` once for every published cache repository. It uses
Git's ordinary reachability and garbage-collection behavior. It does not reconnect shallow history or impose a custom
retained-object set.

Success:

```json
{"status": "ok", "message": {}, "compacted": ["<GIT-URL>", "..."]}
```

## Handleable errors

| Code | Cause and next step |
|---|---|
| `cache.repository.unavailable` | A cached repository path is invalid, unusable, or a Git maintenance operation failed. Inspect the cache configuration and retry. |

## Installation context

`cache` commands operate on the user-level cache and do not use `--installation-context`.
