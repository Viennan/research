# `skills`

`skills` installs the bundled `doctidex-git` Twin Skill into a target directory.

See [common.md](common.md) for shared interface and errors.

The installed skill is version-matched to the CLI that installed it. **After updating the CLI, reinstall the matching
skill with `doctidex-git skills install --path <DEST>`.** See [overview.md](overview.md#prerequisites) for release and
wheel URL discovery.

## Install

```bash
doctidex-git skills install --path <DEST>
```

`--path` is required and names the directory that will contain skills, for example a repository's `.agents/skills`.
The command copies the bundled `doctidex-git` Twin Skill into `<DEST>/doctidex-git/`, replacing the contents of that skill
directory while leaving sibling directories untouched.

`skills install` is a user-level distribution command. It does not resolve a Git root, open `.doctidex-git/`, or use
`--installation-context`.

Success:

```json
{"status": "ok", "message": {}, "skills": ["doctidex-git"], "install-path": "<DEST>/doctidex-git"}
```

## Handleable errors

| Code | Cause and next step |
|---|---|
| `skills.install.target.unavailable` | `--path` is an existing file, cannot be created, or cannot receive files. |
| `skills.install.unavailable` | The bundled Twin Skill tree is missing or cannot be copied. |

## Installation context

`skills install` does not use `--installation-context`.
