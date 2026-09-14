# `validate`

`validate` read-only checks the work model, cross-boundary Markdown links, and managed Worktrees.

See [common.md](common.md) for shared interface and errors.

## Usage

```bash
doctidex-git validate \
  [--subdir <REPOSITORY-PATH> | --only-model-structure]
```

`--subdir` and `--only-model-structure` are mutually exclusive.

The default invocation validates the full work model and scans cross-boundary Markdown links from the Git root.

| Mode | Behavior |
|---|---|
| default | Validate the full work model and scan cross-boundary links from `/`. |
| `--subdir <REPOSITORY-PATH>` | Validate the full work model but scan cross-boundary links only under the requested repository-internal directory. |
| `--only-model-structure` | Validate only the work-model structure; skip the Markdown and link scan. |

`--subdir` narrows the scan, not the work-model check.

## Result

Diagnostics are not execution failures. `validate` returns `status: "ok"` with `valid: false` when diagnostics exist.

```json
{
  "status": "ok",
  "message": {},
  "valid": false,
  "scope": {"repos-path": "/work/repository", "subdir": "/"},
  "diagnostics": [
    {
      "rule": "link.annotation.required",
      "path": "/docs/guide.md",
      "line": 12,
      "message": "A cross-boundary link requires a matching doctidex annotation.",
      "details": {
        "link-path": "/external/example/index.md",
        "expected-cross-boundary-point": "/external/example"
      }
    }
  ]
}
```

## Common diagnostics

| Rule | Meaning |
|---|---|
| `work-model.valid` | Workspace, projections, relationships, managed paths, or ignore rules are invalid. |
| `link.target.exists` | Cross-boundary link target does not exist. |
| `link.annotation.required` | Cross-boundary link lacks a matching annotation. |
| `import.link.tracked` | Link crosses an untracked Installation. |
| `worktree.clean` | Managed Worktree has uncommitted changes. |
| `installation.worktree.dirty` | An Installation has uncommitted changes; reported inside `work-model.valid`. |

`validate` scans Markdown links only when they cross a BoundaryPoint. It does not report path-conformance or
target-existence diagnostics for ordinary local links.

When a tracked Installation's physical directory is absent, validation does not restore it or require its link
targets. This is expected after cloning tracked metadata; use `import query` to see `restore-state` and
`import restore --install-id <INSTALL-ID>` to recreate the physical worktree.

## Cross-boundary annotation

```markdown
[External](/external/example/index.md)
<!-- doctidex: {cross-boundary-point: /external/example} -->
```

## Handleable errors

| Code | Cause and next step |
|---|---|
| `validation.scope.unavailable` | `--subdir` is unreadable, workspace-internal, or crosses a boundary. |
| `validation.scan.unavailable` | Validation scope cannot be traversed or read. |

`validate` never repairs state. Use [`repair`](repair.md) to align recoverable physical objects.

## Installation context

`validate` is allowed when the selected Git root is inside a managed Installation. If `--installation-context`
selects a tracked Installation that is not restored, the command fails with `installation.restore.required`.
