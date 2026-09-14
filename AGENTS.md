# Research Repository Guide

## Purpose

This repository is a research workspace. Each investigation studies an external object — a source repository, a
technical blog post, a paper, or similar material — and produces notes that stay readable and traceable on their own.

It remains an ordinary Git repository. External Git revisions are imported read-only with `doctidex-git` and pinned to
a single commit, while the research writing lives in this repository's own history.

## Layout

```text
.
├── AGENTS.md
├── .agents/skills/          # repository-local Agent Skills
├── .doctidex-git/           # doctidex-git workspace: Installations, Refs, runtime state
└── subjects/                # research documents, one directory per subject
    ├── AGENTS.md
    └── <subject>/           # one subject: its documents and the sources it pins
```

An `AGENTS.md` file in a subdirectory is a **scoped-AGENTS.md**: the root `AGENTS.md` scoped to that path. Read every
**scoped-AGENTS.md** on the path you access before acting.

## Sources

- Import an external Git revision with `doctidex-git import install`. Installed content is read-only and pinned to a
  fixed commit; the CLI's Twin Skill owns those commands, so follow it instead of driving Git by hand.
- Pin a source with a Ref inside the subject that uses it: `subjects/<subject>/sources/<name>`, using the lowercase
  hyphenated source name. The read-only copy then sits beside the documents that depend on it, and two subjects that
  rely on the same source each get their own Ref to the same Installation.
- Record every source in the subject's source index: kind, identifier, fixed revision or URL, and the date it was read.
  Notes stay traceable after an Installation moves to a newer revision.
- A link into `sources/` crosses a boundary point and needs an annotation directly after it. Write the annotation in the
  same form as the link — relative link, relative annotation — because it must be a path prefix of the link as written:

  ```markdown
  [Source](sources/<name>/path/to/doc.md)
  <!-- doctidex: {cross-boundary-point: sources/<name>} -->
  ```

- A link written from the repository root (`/subjects/...`) takes the same absolute form in its annotation, and a link
  into another subject names that subject's Ref. The same rule applies below any declared custom boundary.
  `doctidex-git validate` reports a missing, misplaced, or mismatched annotation as `link.annotation.required`.
- Blogs, papers, and other non-Git material cannot be installed. Cite them by URL and access date, and keep a local
  copy only when a finding depends on that exact artifact.

## Agent Scaffolding

- Read [research-scaffolding-builder](.agents/skills/research-scaffolding-builder/SKILL.md) before changing
  `AGENTS.md`, a **scoped-AGENTS.md**, or anything under `.agents/skills/`. That Skill owns these artifacts and decides
  how they are named and structured.
- Read the [doctidex-git Twin Skill](.agents/skills/doctidex-git/SKILL.md) when a task imports or inspects an external
  revision, creates or checks a Ref or Worktree, or validates the work model. The Twin Skill is installed by the CLI
  and is not owned by this repository: refresh it with `doctidex-git skills install --path .agents/skills` instead of
  editing it.
- Skills this repository owns are named `<repository>-<skill>`. Here the repository name is `research`, so
  `research-scaffolding-builder` maintains itself under the same rule.

## Language

Agent scaffolding — `AGENTS.md`, **scoped-AGENTS.md**, and Skills — is written in English.
