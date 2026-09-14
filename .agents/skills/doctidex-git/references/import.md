# `import`

`import` manages fixed-revision Installations and managed Refs.

See [common.md](common.md) for shared interface and errors.

Installation directories are read-only. Use [`worktree create`](worktree.md) when you need to branch, modify, or commit from an Installation's recorded commit.

## Install

```bash
doctidex-git import install \
  (--tracked | --untracked) \
  --url <GIT-URL> \
  (--branch <BRANCH> | --tag <TAG> | --commit <HASH>) \
  [--key <QUERY-KEY>]...
```

`--tracked` or `--untracked` is required. Exactly one revision selector is required.

Tracking is one-way. `--untracked` applies only to a new Installation; re-running `import install --untracked` on an
existing tracked Installation with a changed branch or tag commit keeps it tracked. To install that selector as
untracked, remove the tracked Installation first.

`--key` is repeatable and adds user query keys to the Installation. The CLI also derives default keys from the Git
URL: the repository path without scheme and without a trailing `.git`, plus `repository@branch` and `repository@tag`
when those selectors are present. Use `import query --key <FRAGMENT>` later to find an Installation by any of its
stored keys.

For example:

```text
https://github.com/example/project.git  -> repository: example/project
git@github.com:example/project.git      -> repository: example/project
```

An `https://` URL is converted to SSH form before keys are derived, so both inputs above produce the same repository
portion. Installing either form with `--branch main` therefore derives the default keys
`example/project` and `example/project@main`.

`--url` accepts a Git SSH URL. An `https://` URL is accepted for convenience and is converted to SSH form at
argument parsing.

Branch and tag selectors resolve once. Re-running the same branch/tag with an unchanged remote commit reuses the Installation; a changed commit replaces it.

The selector path includes a selector-kind segment: `branch/<BRANCH>`, `tag/<TAG>`, or `commit/<HASH>`. For a direct
commit Installation, `install-path` is the shared Installation worktree for that commit. For a branch or tag
Installation, `install-path` is a symbolic link to that shared worktree; commands continue to use the reported path,
and the symlink does not change Installation read-only behavior. See [layout.md](layout.md) for the stable
selector-derived directory organization.

Success:

```json
{
  "status": "ok",
  "message": {},
  "install-id": "<INSTALL-ID>",
  "install-path": "/.doctidex-git/imports/<DOMAIN>/<REPOSITORY>/<branch|tag|commit>/<VALUE>"
}
```

## Restore and track

```bash
doctidex-git import restore --install-id <INSTALL-ID>
doctidex-git import track --install-id <INSTALL-ID>
```

`restore` accepts only tracked Installations and uses the recorded `commit-hash`; it does not re-resolve a branch or tag. `track` promotes an untracked Installation to tracked.

## Ref management

```bash
doctidex-git import ref \
  --install-id <INSTALL-ID> \
  [--src-sub-dir <INSTALL-REPOSITORY-PATH>] \
  --target-dir <REPOSITORY-PATH>

doctidex-git import unref --target-dir <REPOSITORY-PATH>
```

`ref` creates a relative symlink and derives an `import-ref` boundary. `--target-dir` must not be inside `/.doctidex-git/imports/` or `/.doctidex-git/worktrees/`, and must not be below an existing boundary point. Creating a Ref promotes its Installation to tracked. `unref` is a no-op when no Ref exists and is blocked while a Markdown link crosses the Ref boundary.

## Query

```bash
doctidex-git import query \
  (--install-id <INSTALL-ID> | --install-path <REPOSITORY-PATH> | \
   --ref-path <REPOSITORY-PATH> | --key <QUERY-KEY>...)
```

Exactly one selector class is required. `--key` is repeatable fuzzy search. A query key can be any substring of a
stored Installation key; results are ordered by the number of matched keys and then by exact key matches.

`restore-state` is one of `available`, `restore-required`, or `missing`. A tracked Installation with only
metadata and no physical worktree is `restore-required`; run `import restore --install-id <INSTALL-ID>` to make
it available. An untracked Installation with a missing physical path is `missing`.

```json
{
  "status": "ok",
  "message": {},
  "candidates": [
    {
      "git-url": "<GIT-URL>",
      "commit-hash": "<HASH>",
      "install-id": "<INSTALL-ID>",
      "install-path": "/<INSTALL-PATH>",
      "restore-state": "available",
      "keys": ["<QUERY-KEY>"],
      "branch": "<BRANCH-OR-EMPTY>",
      "tag": "<TAG-OR-EMPTY>",
      "refs": [
        {"src-sub-dir": "/<INSTALL-SUB-DIR-OR-EMPTY>", "target-dir": "/<REPOSITORY-PATH>"}
      ]
    }
  ]
}
```

## Remove

```bash
doctidex-git import remove \
  (--install-id <INSTALL-ID> | --untracked | --auto | \
   --presentation-installation-context | --branch <BRANCH>...)
```

`--install-id` selects one Installation. `--untracked` selects all untracked Installations. `--auto` selects untracked Installations and Installations without managed Refs.
`--presentation-installation-context` selects InstallationShares that can only be Presentation-Installation shares:
`install-ids` is empty and `branch-refs` is empty or contains only the current branch. `--auto` also includes this
cleanup.

`--branch` is repeatable and selects branch snapshots by branch short name. The currently checked-out branch cannot be
selected; an unknown branch name is a no-op.

`--auto` additionally removes branch snapshots whose branch no longer exists locally. The current branch is never
removed by `--auto`.

Removal is blocked when a tracked Installation still has a Ref or an in-scope Markdown link crosses its boundary.

Removing an Installation removes its share membership. The shared physical worktree survives while another branch
still references the same share.

Removing a branch snapshot also removes that branch from `InstallationShare.branch-refs`. When a share has no
remaining `install-ids`, `context-references`, or `branch-refs`, the share and its physical worktree are deleted.

## Unload

```bash
doctidex-git import unload --install-id <INSTALL-ID>...
```

`--install-id` is repeatable and at least one is required. `unload` accepts only tracked Installations and detaches
them from their Installation share while keeping the Installation record. A tracked Installation that is already
`restore-required` is a no-op.

Unloading a branch or tag Installation removes its selector symlink. A direct-commit Installation's physical checkout
is removed only when its share has no remaining `install-id`, `context-reference`, or `branch-ref`; when none remain,
the share and its physical worktree are deleted.

Managed Refs and cross-boundary Markdown links are left in place. After unload, `import query` reports
`restore-required`, and `import restore --install-id <INSTALL-ID>` recreates the physical checkout.

## Handleable errors

| Code | Cause and next step |
|---|---|
| `revision.unresolvable` | Selector cannot resolve to a commit. |
| `cache.repository.unavailable` | Bare repository cannot be obtained. |
| `installation.target.unavailable` | Install path is occupied or unusable. |
| `installation.not-found` | Requested Installation does not exist. |
| `installation.tracking-state.invalid` | `restore` or `unload` received an untracked Installation. |
| `installation.restore.unavailable` | Tracked Installation cannot be restored at its commit. |
| `installation.restore.required` | A tracked Installation is not restored; run `import restore`. |
| `installation.remove.blocked` | Ref or cross-boundary link still depends on the Installation. |
| `installation.unload.reconcile.failed` | A selector symlink, share, or physical worktree could not be removed during unload. |
| `import.branch-snapshot.remove.current-branch` | `--branch` named the currently checked-out branch. |
| `import.branch-snapshot.reconcile.failed` | A share or physical worktree could not be cleaned during snapshot removal. |
| `ref.source.unavailable` | Installation or `src-sub-dir` cannot be a link source. |
| `ref.target.unavailable` | Target is occupied, is under a managed directory, is below an existing boundary point, or cannot be created. |
| `ref.target.inconsistent` | Physical symlink does not match the Ref record. |
| `ref.remove.blocked` | Markdown link still crosses the Ref boundary. |

## Installation context

To run an import command against an Installation, pass its owner-side `install-id`:

```bash
doctidex-git --repos-path <OWNER-ROOT> --installation-context <INSTALL-ID> import query ...
doctidex-git --repos-path <OWNER-ROOT> --installation-context <INSTALL-ID> import restore --install-id <INSTALL-ID>
```

In Installation context, `import query` and `import restore` are allowed. `import restore` reads the local
Installation and publishes an owner-side `presentation-path` and `presentation-install-id` without mutating that
Installation's own work model. `import install`, `import track`, `import ref`, and `import unref` are forbidden.
`import remove` and `import unload` are not available in Installation context.

Running an import command with a path inside a managed Installation but without `--installation-context`
produces `installation.context.argument-required`.

When an Installation-context `import query` result has no `presentation-path`, the Installation has not yet been
published into the owner work model. In the same Installation context, run `import restore --install-id
<INSTALL-ID>` to obtain its owner-side path and `presentation-install-id`. Use that `presentation-install-id` as the
next `--installation-context <INSTALL-ID>` for recursive access.

In Installation-context query results, a candidate with `presentation-path` is `available` and also includes
`presentation-install-id`; otherwise `restore-state` falls back to the local `install-path` check.
