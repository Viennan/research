# Common Interface and Recovery

All command clusters share Git-root selection, repository paths, cache configuration, JSON output, and recovery boundaries.

## Git root and paths

Commands shown as `doctidex-git` assume the installed entry point is resolvable. See
[overview.md](overview.md#prerequisites) for release discovery and installation.

Print the installed CLI version with:

```bash
doctidex-git --version
```

Every command accepts:

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] [--installation-context <INSTALL-ID>] <command> [options]
```

When `--repos-path` is omitted, the CLI discovers the enclosing Git root. When provided, it must name the Git root exactly.
When `--installation-context` is supplied and `--repos-path` is omitted, the CLI treats the sole ancestor
`.doctidex-git` owner as the effective owner root; zero or multiple candidates fail before mutation.

Repository-internal paths begin with `/` and are rooted at the Git root, not the host filesystem:

```text
/docs/guide.md
/.doctidex-git/imports/...
```

Path normalization may collapse `.` and `..`, but the result must not escape the repository.

## Cache configuration

The default doctidex-git home is `~/.doctidex-git`. `DOCTIDEX-GIT-HOME` selects another home. The global config is
`<home>/config.toml` and is created empty on first use.

`config.toml` may set `cache-path`; otherwise the cache is `<home>/cache`. A Git-root command also reads
`.doctidex-git/config.toml` from the selected repository, and that file may override `cache-path`. User-level
`cache clean` and `cache compact` use the global config only. A relative `cache-path` is resolved from the directory
that contains the `config.toml` which declared it.

Inside the cache root, `cache-status.json` records published cache entries, and bare Git repositories live under
`data/<domain>/<repository...>`. The cache is not the authority for Installations, Refs, or Worktrees. Use
[`cache clean`](cache.md#clean) and [`cache compact`](cache.md#compact) to maintain the cache.

## Installation context

`--installation-context <INSTALL-ID>` explicitly selects an Installation already recorded in the owner work model.
When it is supplied, the CLI runs the command in Installation context for that `install-id`; the owner root comes
from `--repos-path` or the ancestor-owner rule above.

Installation context is meaningful only when the selected Installation itself is a doctidex-git-managed
repository with a local `.doctidex-git` work model. If the Installation is an ordinary external checkout, the
context commands that read Installation-local declarations fail with `installation.context.unavailable`.

When `--installation-context` is omitted, path detection is defensive only. A selected root that matches a recorded
`Worktree.work_path` is an ordinary repository path. If the path otherwise appears to be inside a managed
Installation, the command fails with `installation.context.argument-required` and does not run.

### Allowed commands

The following commands are allowed in Installation context:

- `validate`
- `boundary-set parse`
- `import query`
- `import restore`

`import restore` has special Installation-local routing: it reads the requested Installation from the local work model
and publishes an owner-side `presentation-path` and `presentation-install-id` without mutating that Installation's own
work model. It never creates a nested Installation inside the current Installation. The returned Installation keeps
its local identity.

If the selected `--installation-context` refers to a tracked Installation whose physical worktree is absent, the
command fails with `installation.restore.required` before mutation. Run `import restore --install-id
<INSTALL-ID>` in the owner root to recreate the physical worktree.

When an Installation-context `import query` result has no `presentation-path`, the Installation exists only in the
local work model. In the same Installation context, run `import restore --install-id <INSTALL-ID>` to obtain its
owner-side path and `presentation-install-id`. Use that `presentation-install-id` as the next
`--installation-context <INSTALL-ID>` for recursive access.

### Forbidden commands

The following commands are forbidden inside an Installation:

- `init`
- `worktree`
- `import install`
- `import track`
- `import ref`
- `import unref`
- `boundary-set add`
- `boundary-set remove`
- `repair`
- `hook`

Other commands are not yet available in Installation context.

### Context errors

| Code | Meaning |
|---|---|
| `installation.context.argument-required` | The path appears to be inside an Installation; pass `--installation-context <INSTALL-ID>`. |
| `installation.context.owner-required` | `--installation-context` was supplied, but no owner `.doctidex-git` candidate could be found from the current path. |
| `installation.owner.ambiguous` | The path is nested in multiple Installation owner workspaces. |
| `installation.context.forbidden` | The requested command is prohibited inside an Installation. |
| `installation.context.unavailable` | The command is not yet available inside an Installation, or local declarations cannot be read. |
| `installation.restore.required` | The selected tracked Installation has not been restored; run `import restore`. |

## Success results and exit codes

Except for `validate`, a generic success result is:

```json
{"status": "ok", "message": {}}
```

Command-specific fields are documented by each command cluster.

| Result | `status` | Exit code |
|---|---|---:|
| Command completed | `ok` | 0 |
| `validate` found diagnostics | `ok`, `valid: false` | 1 |
| Argument, model, or workflow failure | `error` | 2 |

## Structured errors

Failures return a stable machine-readable envelope:

```json
{
  "status": "error",
  "message": {
    "code": "<STABLE-CODE>",
    "summary": "<HUMAN-READABLE-SUMMARY>",
    "context": {"command": "<COMMAND>", "repos-path": "<GIT-ROOT>"},
    "subject": {"kind": "<OBJECT-KIND>"},
    "details": {}
  }
}
```

Automation must rely on `message.code` and `message.details`, not `summary`.

Common errors:

| Code | Cause and next step |
|---|---|
| `argument.invalid` | Missing, conflicting, duplicate, or malformed arguments. |
| `git-root.unresolved` | The requested or discovered path is not a usable Git root. |
| `repository-path.invalid` | A repository-internal path is malformed or escapes the repository. |
| `work-model.uninitialized` | Run `init` first. |
| `work-model.invalid` | Run `validate --only-model-structure`. |
| `store.transaction.unavailable` | Store or residual transaction recovery failed; run `repair` or retry. |

## Recovery boundary

`validate` is read-only. `repair` aligns managed physical objects with JSON records but does not modify Markdown links or roll back Git history.

Normal commands detect residual RuntimeStore journals, run internal repair, and retry up to three times. For direct investigation:

```bash
doctidex-git validate --only-model-structure
doctidex-git repair
doctidex-git validate
```
