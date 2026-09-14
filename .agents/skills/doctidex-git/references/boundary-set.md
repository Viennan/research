# `boundary-set`

`boundary-set` manages custom escape boundaries and resolves the first BoundaryPoint for paths.

See [common.md](common.md) for shared interface and errors.

## Add and remove

```bash
doctidex-git boundary-set add --path <REPOSITORY-PATH> [--path <REPOSITORY-PATH>]...
doctidex-git boundary-set remove --path <REPOSITORY-PATH> [--path <REPOSITORY-PATH>]...
```

`--path` is required and repeatable. Paths are repository-internal absolute paths; the directory need not exist.

`add` stores custom records in `boundary-set.json`. `remove` removes only custom records and is a no-op when absent. Derived boundaries are managed by their owning commands.

## Parse

```bash
doctidex-git boundary-set parse --path <REPOSITORY-PATH> [--path <REPOSITORY-PATH>]...
```

Example result:

```json
{
  "status": "ok",
  "message": {},
  "results": [
    {
      "path": "/external/example/readme.md",
      "has-boundary": true,
      "boundary-point": "/external/example",
      "boundary-type": "import-ref"
    }
  ]
}
```

`boundary-type` is `custom`, `import`, `import-ref`, or `worktree`.

## Derived boundaries

| Type | Managed by |
|---|---|
| `custom` | `boundary-set` |
| `import` | `import` |
| `import-ref` | `import ref` / `import unref` |
| `worktree` | `worktree` |

## Handleable errors

| Code | Cause and next step |
|---|---|
| `boundary-point.remove.prohibited` | `remove` targeted a derived boundary; use its owning command. |

## Installation context

`boundary-set parse` is allowed inside a managed Installation. `boundary-set add` and `boundary-set remove` are forbidden there.
