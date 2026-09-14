---
name: doctidex-git
description: Twin agent skill for doctidex-git. Use it to turn a Git repository into a node in an interconnected knowledge network — navigate docs, reference fixed external revisions, and stay a normal Git repo.
doctidex:
  version: 2.2.2
---

# doctidex-git

This is the Twin Skill for `doctidex-git`.

This is a guide, not a script. It shows the CLI's capability space and points to
[overview.md](references/overview.md) and the other `references/` documents for authoritative detail.

## Install and update

This skill is authoritative only for the CLI version recorded by `doctidex.version`.

- Get the CLI version and wheel URL from the `doctidex` GitHub Releases at
  https://github.com/Viennan/doctidex/releases.
- Recommended: `pipx install --force <WHEEL-URL>`.
- Without pipx, install the wheel with `pip` or `uv` in a virtual environment and use
  `<venv>/bin/doctidex-git`; see [overview.md](references/overview.md#prerequisites).
- **After updating the CLI, or whenever `doctidex-git --version` does not match `doctidex.version`, reinstall the
  matching skill with `doctidex-git skills install --path <DEST>`.**

## Before you start

- Work from inside a Git repository, or pass `--repos-path <REPOSITORY-ROOT>`.
- Before use, especially after updating the CLI or reinstalling skills, run `doctidex-git --version` and compare it
  with `doctidex.version`.
- Have `rg` available when possible; the CLI falls back to a slower Python scan when it is missing.
- Read [overview.md](references/overview.md) to establish the shared mental model: Git root, Installation, Ref,
  Worktree, BoundaryPoint, and StructuredLinkAnnotation.
- Treat `/.doctidex-git/imports/` and `/.doctidex-git/worktrees/` as searchable directories; when a domain,
  repository, or selector is already known, search the derived path directly. See [layout.md](references/layout.md)
  for the path contract.

## Capability space

`doctidex-git` helps a repository participate in an interconnected knowledge network while remaining an ordinary Git
repository. The command clusters below are independent capabilities; use the ones the current task needs.

| Area | What it gives you | Start here |
|---|---|---|
| Workspace | `init` creates `.doctidex-git/`; `validate --only-model-structure` checks the model shape. | [init.md](references/init.md), [validate.md](references/validate.md) |
| Installations | `import install`, `restore`, `track`, `query`, `remove`, and `unload` manage fixed external revisions and restore state. | [import.md](references/import.md) |
| Refs | `import ref` and `import unref` expose stable read-only paths into Installations. | [import.md](references/import.md) |
| Worktrees | `worktree create`, `query`, and `remove` manage editable worktrees from an Installation or URL. | [worktree.md](references/worktree.md) |
| Boundaries | `boundary-set add`, `remove`, and `parse` declare or inspect custom boundary points. | [boundary-set.md](references/boundary-set.md) |
| Validation and repair | `validate` observes problems; `repair` aligns recoverable physical state. | [validate.md](references/validate.md), [repair.md](references/repair.md) |
| Hooks | `hook install`, `pre-commit`, and `post-checkout` keep runtime state branch-consistent and validate before commit. | [hook.md](references/hook.md) |
| Cache | `cache clean` and `cache compact` maintain the user-level Git cache. | [cache.md](references/cache.md) |
| Skills | `skills install` publishes the bundled Twin Skill into a target repository. | [skills.md](references/skills.md) |
| Installation context | `--installation-context <INSTALL-ID>` selects a doctidex-managed Installation and exposes its local work model to a restricted command set; restored results expose `presentation-install-id` for recursive use. | [common.md](references/common.md#installation-context) |

These areas compose. For example, an Installation plus a Ref creates a stable external path; a Worktree can then make
an editable copy of that content; validation and repair keep the model and physical state aligned.

## Common scenarios

These examples show common combinations; they are not the only supported workflows.

### Bootstrap a repository

For a repository that does not yet have `.doctidex-git/`, a common start is:

```bash
doctidex-git init
doctidex-git validate --only-model-structure
```

For a fresh clone, `.doctidex-git/` may already exist while local Git hooks do not. Reinstall hooks and verify:

```bash
doctidex-git hook install
doctidex-git validate --only-model-structure
```

See [init.md](references/init.md), [hook.md](references/hook.md), and [validate.md](references/validate.md).

### Reference a fixed external revision

When a document needs content from another repository at a known commit, install the revision and expose it with a
Ref:

```bash
doctidex-git import install \
  --tracked \
  --url <GIT-URL> \
  --branch <BRANCH> \
  --key <QUERY-KEY>

doctidex-git import ref \
  --install-id <INSTALL-ID> \
  [--src-sub-dir /<INSTALL-REPOSITORY-PATH>] \
  --target-dir /external/<NAME>
```

Use `--tracked` when the reference should persist across clones; use `--untracked` for a temporary reference. A
branch or tag resolves once to a commit, so run `import install` again only when that fixed revision should change.
Tracking is one-way: re-running `--untracked` on an existing tracked Installation does not demote it; remove it first
to install that selector as untracked.
`--key` is repeatable; the CLI also derives default keys from the repository path and selector, so fuzzy lookups can
start from [those defaults](references/import.md#install) or from keys you add.
In a fresh clone, a tracked Installation may be metadata-only until restored:

```bash
doctidex-git import restore --install-id <INSTALL-ID>
```

Find an Installation later with a fuzzy key fragment:

```bash
doctidex-git import query --key <QUERY-KEY-FRAGMENT>
```

See [import.md](references/import.md) for selectors, Ref rules, removal, and `restore-state`.

### Free a tracked checkout without losing it

`import unload` detaches a tracked Installation from its shared checkout while keeping its record:

```bash
doctidex-git import unload --install-id <INSTALL-ID>
```

`import query` then reports `restore-required`; use `import restore --install-id <INSTALL-ID>` to recreate it.

### Modify external content safely

An Installation is read-only. Create an editable Worktree when the task needs to branch, modify, or commit from its
recorded commit:

```bash
doctidex-git worktree create \
  --install-id <INSTALL-ID> \
  --work-path /projects/<NAME>
```

See [worktree.md](references/worktree.md).

### Link across a boundary

A Markdown link that crosses a BoundaryPoint must be followed by a `doctidex` StructuredLinkAnnotation naming the first
boundary crossed:

```markdown
[External](/external/<NAME>/path/to/doc.md)
<!-- doctidex: {cross-boundary-point: /external/<NAME>} -->
```

Paths beginning with `/` are rooted at the Git root; other paths are relative to the source document. To check a
smaller scope:

```bash
doctidex-git validate --subdir <REPOSITORY-PATH>
```

See [overview.md](references/overview.md), [common.md](references/common.md), and
[validate.md](references/validate.md).

### Add a custom boundary

When an ordinary vendored or local directory should stop the link scan, declare it as a custom BoundaryPoint:

```bash
doctidex-git boundary-set add --path /vendor/<NAME>
doctidex-git boundary-set parse --path /vendor/<NAME>/readme.md
```

See [boundary-set.md](references/boundary-set.md).

### Operate inside a managed Installation context

When the selected repository is itself a doctidex-managed Installation, prefer passing both the owner root and the
selected Installation:

```bash
doctidex-git --repos-path <OWNER-ROOT> --installation-context <INSTALL-ID> import query
```

`validate`, `boundary-set parse`, `import query`, and `import restore` are allowed in that context; other mutating
commands are rejected. After restore, use the returned `presentation-install-id` as the next
`--installation-context <INSTALL-ID>` for recursive access. See
[common.md](references/common.md#installation-context).

If an indirect child Installation is valuable enough to reference directly from the owner repository, return to the
owner root and promote it to a direct Installation with `import install`, then reference it like any other
Installation.

### Check and repair state

`validate` observes problems; `repair` aligns recoverable physical state. These are separate capabilities, so use the
one that fits the current need:

```bash
doctidex-git validate
doctidex-git repair
```

`repair` does not rewrite Markdown links or roll back Git history. See [validate.md](references/validate.md) and
[repair.md](references/repair.md).

## Rules you must follow

- Never edit the state JSON under `.doctidex-git/` by hand.
- Treat Installation directories as read-only; create a Worktree when a revision must be modified.
- Never remove or rename managed Installation, Worktree, Ref, or boundary paths by hand.
- Always place a `doctidex` StructuredLinkAnnotation immediately after a Markdown link that crosses a BoundaryPoint.
- Use `validate` to observe problems; use `repair` to recover physical state, not to rewrite document content.

For shared result envelopes, structured errors, and Installation-context behavior, use
[common.md](references/common.md).
