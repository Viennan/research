# Repository layout

This reference describes the stable directory organization that `doctidex-git` creates for Installations and
default Worktrees. It is a lookup contract for locating installed and editable content on disk; use
[overview.md](overview.md) for the mental model and command-level behavior.

The managed directories are searchable by source:

```text
/.doctidex-git/imports/      # read-only Installations
/.doctidex-git/worktrees/    # default editable Worktrees
```

The layout is an inventory for locating content, not a replacement for the CLI's state records.

## Installation layout

An Installation path is selector-derived:

```text
/.doctidex-git/imports/<domain>/<repository-path...>/<selector-kind>/<selector-value...>
```

- `<domain>` is the Git URL hostname, or `local` for non-host sources.
- `<repository-path...>` is the parsed repository path without a leading slash or trailing `.git`; it may contain
  more than one component.
- `<selector-kind>` is exactly `branch`, `tag`, or `commit`.
- `<selector-value...>` may contain slashes, for example a branch named `feature/search`.

For `branch` and `tag`, the selector path is a symbolic link to the shared commit checkout:

```text
/.doctidex-git/imports/<domain>/<repository-path...>/commit/<commit-hash>
```

A direct `commit` Installation uses that commit checkout path itself. A reader can follow either the selector path
or, when the commit is known, the `commit/` path to reach the same physical content.

The physical directory exists only while the Installation is restored. Search the derived path when the content is
known to be present; `import query` reports `restore-state`.

## Worktree layout

A default Worktree path follows the same source hierarchy:

```text
/.doctidex-git/worktrees/<domain>/<repository-path...>/<tree-name...>
```

`<tree-name...>` comes from `--tree-name` or from the short random name chosen when no name is supplied. A Worktree
created with `worktree create --work-path` has no derivable default path and must be found with
[`worktree query`](worktree.md#query-and-remove).

## Search directly

When a domain, repository, or revision selector is already known, search the derived path directly instead of first
asking the CLI for the path. Fall back to the query commands when the exact record, identity, restore state, Ref, or
custom path matters:

- [`import query`](import.md#query) for Installation identity, `install-path`, Refs, and `restore-state`.
- [`worktree query`](worktree.md#query-and-remove) for a recorded Worktree path.

## Read and edit boundaries

The layout locates content but does not change its ownership rules:

- Installation directories are read-only; create a Worktree when a revision must be modified.
- Never edit state JSON under `.doctidex-git/` by hand.
- Never remove or rename managed Installation, Worktree, Ref, or boundary paths by hand.

See [overview.md](overview.md#usage-boundaries) for the complete usage boundaries.
