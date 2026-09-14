# Research Documents

Scoped guidance for everything under `subjects/`. The root `AGENTS.md` still applies.

## Subjects

```text
subjects/
└── <subject>/
    ├── README.md            # question or scope, status, source index
    ├── sources/             # this subject's Refs into imported external revisions
    └── <angle>.md           # findings, comparisons, reproductions, open questions
```

- One directory per subject: `subjects/<subject>/`, lowercase hyphenated, named for the object or question under study
  rather than the tool used to study it.
- `README.md` in a subject directory is its entry point. It states the question or scope, the current status
  (`planned`, `active`, `paused`, or `done`), and that subject's source index.
- `sources/` holds the Refs this subject reads: one directory per pinned source, created through the Twin Skill. It is
  the only managed path inside a subject directory; create it on first use and never edit inside it.
- Further documents sit beside it, named for the angle they cover — findings, comparisons, reproductions, open
  questions.

## Source Index

- List every source the subject used, imported Installations and external URLs alike, with its identifier and fixed
  revision or URL plus the date it was read.
- Cite the specific source path or URL for each claim that depends on it. A citation into `sources/...` follows the
  boundary annotation rule in the root `AGENTS.md`.

## Imported Sources

- Installation and Ref content is read-only. To build, run, patch, or otherwise experiment with imported code, create
  a Worktree through the Twin Skill rather than editing the Installation, the Ref path inside the subject directory, or
  the installed content.
- Managed Worktrees live under `.doctidex-git/worktrees/`, outside the subject directory. Copy a result into the
  subject directory only when a finding depends on it.

## Writing

- Write for a reader who does not have the source open: state what was checked, what held, and what did not.
- Keep evidence and interpretation distinguishable, and mark anything inferred, unverified, or already stale.
- Summarize rather than pasting long verbatim excerpts, and quote only what a finding needs.
- Prefer one document per question over one document per browsing session.
