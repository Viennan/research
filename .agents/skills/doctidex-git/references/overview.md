# doctidex-git v2 User Guide

`doctidex-git` organizes Git repositories into an interconnected knowledge network. A repository remains ordinary source
and history while also becoming a navigable knowledge base that can reference fixed external Git revisions and can itself
be referenced by other repositories.

The tool manages:

- fixed-revision external repositories as Installations;
- managed symbolic Refs into Installations;
- editable Worktrees;
- custom boundary points;
- validation and repair;
- Git hooks that keep untracked runtime state consistent across branch switches and validate work-model structure
  before commits;
- installable Twin Skills that carry the supported `doctidex-git` workflow into another repository.

This guide states the user-facing behavior of `doctidex-git`; internal design details are intentionally out of scope
here.

## Prerequisites

- Linux or macOS.
- A Git worktree.
- Supported CLI versions and wheel URLs are published in the `doctidex` GitHub Releases at
  https://github.com/Viennan/doctidex/releases. Use the wheel URL from the release you want.
- Recommended: `pipx install --force <WHEEL-URL>`.
- Without pipx: create a virtual environment, install the wheel with `pip` or `uv`, then use
  `<venv>/bin/doctidex-git`.
- **After updating the CLI, reinstall the matching skill with `doctidex-git skills install --path <DEST>`.**
- `rg` (ripgrep) with PCRE2 support is strongly recommended for fast Markdown link scanning. When `rg` is
  unavailable, `doctidex-git` falls back to a per-file Python scan with the same behavior but reduced performance on
  large repositories.

Run `doctidex-git --version` to confirm the installed CLI version; this informational option prints plain text, while
command results use JSON.

## Quick start

```bash
cd /path/to/repository
doctidex-git init
doctidex-git validate --only-model-structure
```

Then install a fixed external revision and expose it with a Ref:

```bash
doctidex-git import install \
  --tracked \
  --url git@github.com:example/project.git \
  --branch main \
  --key example

doctidex-git import ref \
  --install-id <INSTALL-ID> \
  --target-dir /external/example

doctidex-git validate
```

## Repository layout at a glance

The following directory tree shows one repository with an Installation, a Ref, an editable Worktree, and a custom
boundary. It is a usage illustration, not a required directory-tree standard. The stable selector-derived directory
layout used for direct search is documented in [layout.md](layout.md).

```text
/path/to/repository                       # Git root; /... paths start here
├── README.md
├── docs/
│   └── guide.md
├── .doctidex-git/                        # Workspace
│   ├── config.toml
│   ├── boundary-set.json                 # custom BoundaryPoints
│   ├── imports.json                      # tracked Installations
│   ├── import-refs.json                  # Refs
│   ├── runtime.json                      # untracked Installations, Worktrees, shares, branch snapshots
│   ├── imports/
│   │   └── github.com/
│   │       └── example/
│   │           └── <commit>/             # read-only Installation
│   └── worktrees/
│       └── github.com/
│           └── example/
│               └── <name>/               # editable Worktree
├── external/
│   └── example -> Installation           # Ref; /external/example is an import-ref BoundaryPoint
├── vendor/
│   └── third-party/                      # custom BoundaryPoint
└── links.md
```

The diagram maps to the retained concepts:

- `.doctidex-git/` is the managed Workspace.
- `.doctidex-git/imports/...` holds read-only Installations.
- `.doctidex-git/worktrees/...` is the default location for editable Worktrees; `worktree create --work-path` can place one elsewhere.
- An Installation can contain its own `.doctidex-git/`, which is the source of recursive sub-Installations.
- `external/example` is a Ref; its target path is a derived `import-ref` BoundaryPoint.
- `vendor/third-party` can be declared as a custom BoundaryPoint in `boundary-set.json`.
- A Markdown link that crosses one of those boundaries uses a StructuredLinkAnnotation:

```markdown
[External](/external/example/readme.md)
<!-- doctidex: {cross-boundary-point: /external/example} -->
```

## Mental model

| Concept | User-visible meaning |
|---|---|
| Git root | The boundary for one command and the root of repository-internal `/...` paths. |
| Workspace | `.doctidex-git/` under the Git root. |
| Installation | One external Git URL fixed at one commit and installed at one read-only path. |
| Ref | A managed symbolic link into an Installation. |
| Worktree | An editable Git worktree whose base commit is recorded; it may branch, modify, and commit freely. |
| Restore state | Whether a tracked Installation's physical worktree is present or requires `import restore`. |
| BoundaryPoint | A path where the current repository's link and scan rules stop. |
| StructuredLinkAnnotation | A `doctidex` comment that records the first BoundaryPoint crossed by a Markdown link. |
| Branch snapshot | A branch-specific runtime history used to restore untracked Installation state after checkout. |

## Installation context

Prefer passing both `--repos-path <OWNER-ROOT>` and `--installation-context <INSTALL-ID>` to operate on a recorded
Installation. In that context, `validate`,
`boundary-set parse`, `import query`, and `import restore` are allowed. Commands that create or modify owner-side
Installations, Refs, Worktrees, custom boundaries, hooks, or repair are forbidden. Running from inside a managed
Installation without `--installation-context` is blocked and asks for the argument. `import restore` does not mutate
the selected Installation's own work model; it publishes an owner-side `presentation-path` and
`presentation-install-id`. Use that `presentation-install-id` as the next `--installation-context <INSTALL-ID>` for
recursive access.

This context is meaningful only when the selected Installation is itself maintained by doctidex-git and has its
own `.doctidex-git` work model.

`import query` reports `restore-state`. Tracked Installations are metadata-only until restored; after clone or
physical removal, run `import restore --install-id <INSTALL-ID>`.

See [common.md](common.md#installation-context) for the complete behavior.

## Core behaviors at a glance

| Area | Essential behavior | Detail |
|---|---|---|
| Workspace bootstrap | Run `init` once, then `validate --only-model-structure`. | `init` creates `.doctidex-git/` and its state projections. |
| Installations and Refs | Install a fixed revision, then expose it with a Ref. | Use `import install`, `import ref`, `import restore`, `import query`, `import remove`, and `import unload`. |
| Worktrees | Create an editable worktree from an Installation or URL. | Use `worktree create`, `worktree query`, and `worktree remove`. |
| Custom boundaries | Add or remove custom escape paths. | Use `boundary-set add`, `boundary-set remove`, and `boundary-set parse`. |
| Validation and repair | Observe problems without changes, then repair recoverable physical state. | Use `validate` first; use `repair` to align model and physical objects. |
| Git hooks | Keep untracked runtime state branch-consistent and validate work-model structure before commits. | `init` installs hooks; use `hook install`, `hook pre-commit`, and `hook post-checkout`. |
| Skills | Install the bundled `doctidex-git` Twin Skill into another repository. | Use `skills install --path <DEST>`; see [skills.md](skills.md). |
| Version | Print the installed CLI version as plain text. | Use `doctidex-git --version`. |
| Result contract | Every command emits machine-readable JSON; `--version` is a plain informational option. | Success uses `status: "ok"`; `validate` adds `valid`; failures use stable `message.code`. |
| Cache | The CLI caches bare repositories under a configured cache root. | Configure with `DOCTIDEX-GIT-HOME`, the global `config.toml`, and optional repository `cache-path`; maintain it with `cache clean` and `cache compact`; see [cache.md](cache.md). |
| Installation context | Select an Installation with `--installation-context`; the allowed command set is restricted. | Only `validate`, `boundary-set parse`, `import query`, and `import restore` are allowed; `import restore` returns `presentation-install-id` for the next recursive `--installation-context`. |
| Restore state | Tracked metadata may not have a physical worktree. | `import query` reports `available`, `restore-required`, or `missing`; run `import restore` for `restore-required`, or `import unload` to keep metadata while freeing a checkout. |

## Command documents

| Task | Document |
|---|---|
| Common interface and recovery | [common.md](common.md) |
| Initialize the work model | [init.md](init.md) |
| Manage custom boundaries | [boundary-set.md](boundary-set.md) |
| Manage Installations and Refs | [import.md](import.md) |
| Manage Worktrees | [worktree.md](worktree.md) |
| Validate the model and links | [validate.md](validate.md) |
| Align physical state with the model | [repair.md](repair.md) |
| Install and run Git hooks | [hook.md](hook.md) |
| Maintain the Git cache | [cache.md](cache.md) |
| Install Twin Skills | [skills.md](skills.md) |

## Usage boundaries

Do not edit state JSON under `.doctidex-git/` by hand. Do not commit managed Installation or Worktree directories.
Installation directories are read-only; create a Worktree when you need to modify or commit from a fixed revision. Use
`validate` to observe problems and `repair` to align recoverable physical state.

`hook install` and successful first-time `init` install `pre-commit` and `post-checkout` hooks. The `pre-commit` hook
validates work-model structure before commits, and the `post-checkout` hook keeps untracked Installation and share
state branch-consistent; see [hook.md](hook.md).

`doctidex-git` coordinates only `doctidex-git` processes that follow its lock and transaction protocol. It does not
guarantee race safety against direct external edits, and it never rewrites Git history or undoes commits. `repair` may
discard uncommitted Installation changes but never Worktree changes or commit history; see
[repair.md](repair.md#what-repair-does-not-do).
