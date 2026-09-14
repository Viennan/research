# `init`

`init` establishes the `.doctidex-git` work model.

See [common.md](common.md) for shared interface and errors.

## Usage

```bash
doctidex-git [--repos-path <REPOSITORY-ROOT-PATH>] init
```

## First initialization

When `.doctidex-git/` is absent or empty, `init` creates:

```text
.doctidex-git/
├── config.toml
├── boundary-set.json
├── imports.json
├── import-refs.json
└── runtime.json
```

On successful first initialization, `init` also installs the supported Git hooks, including the `pre-commit` and
`post-checkout` hooks. See [hook.md](hook.md).

## Existing workspace

If `.doctidex-git/` exists and is non-empty, `init` does not overwrite state. It returns:

```json
{
  "status": "ok",
  "message": {
    "code": "workspace.already-initialized",
    "summary": "Initialization has already been run; use validate --only-model-structure to check the work model.",
    "details": {"next-command": "validate --only-model-structure"}
  }
}
```

## Handleable errors

| Code | Cause and next step |
|---|---|
| `workspace.initialize.failed` | Workspace artifacts or ignore rules could not be established. |

`init` does not create Installations, Refs, Worktrees, or custom boundaries.

## Installation context

`init` is forbidden when the selected Git root is inside a managed Installation. See [common.md](common.md) for Installation-context behavior.
