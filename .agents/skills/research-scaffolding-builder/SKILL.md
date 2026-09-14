---
name: research-scaffolding-builder
description: Maintain this repository's Agent scaffolding: the AGENTS.md hierarchy, scoped AGENTS.md files, and the Skills this scaffolding creates and owns. Use before changing any of those artifacts.
---

# Research Scaffolding Builder

This is a guide, not a script. Use judgment to keep the repository's Agent scaffolding coherent, discoverable, and
proportionate to the work it supports.

## Scope

This Skill governs changes to `AGENTS.md`, **scoped-AGENTS.md**, `.agents/skills/`, Skill metadata, and closely
related repository-local Agent scaffolding. Before changing these artifacts, read the applicable `AGENTS.md` files
and the relevant existing Skills.

## Ownership

This Skill owns the scaffolding it creates and maintains. Skills installed by another tool or distributed by another
repository are not part of that scaffolding: they keep their own names, metadata, and update mechanism, and this
Skill does not create, rename, rewrite, or delete them. Read such a Skill when routing or cross-referencing needs it,
and leave its files to whoever installs and updates it.

## Language

Write scaffolding documentation, instructions, and prompts in one consistent language. Follow the language the
repository's Agent guidance already uses; when none exists yet, choose one deliberately and keep new and edited
scaffolding in it.

## Terminology

Use fixed terms for representative, core, or key concepts in `AGENTS.md` files and Skills. Prefer a short bold term,
such as **scoped-AGENTS.md**, over repeating the same explanation.

A root `AGENTS.md` is repository-wide. A **scoped-AGENTS.md** is an `AGENTS.md` placed in a subdirectory to provide
directory-scoped guidance.

Reference a term defined elsewhere with `[term](relative-link)`. For Skills, this is also how one Skill may cite
another Skill's term or guidance.

## AGENTS.md

Use the root `AGENTS.md` for repository-wide identity, stable authority locations, and routing into scoped guidance.
Add a **scoped-AGENTS.md** only when a directory needs durable local rules that cannot remain clear at the root. Keep
scoped files shallow and make the local scope evident from their directory.

Typical scoped guidance includes documentation conventions below a documentation directory, code conventions at a
code root or its parent, and Skill routing that applies to one area only. Let a **scoped-AGENTS.md** supplement or
narrow the applicable parent guidance; explain any intentional exception where it is declared.

When an `AGENTS.md` file, including a **scoped-AGENTS.md**, would contain a large explicit write-oriented workflow,
move that workflow into a dedicated Skill or merge it into an existing Skill serving the same subject. Keep the
`AGENTS.md` as routing and orchestration: state when and how to use the Skill, and provide enough detail beyond the
Skill's frontmatter description to select and sequence the workflow. Keep trigger conditions in the `AGENTS.md`; move
only execution rules, operation steps, and checks into the Skill.

The root `AGENTS.md` must include a repository-structure diagram. Show the trees an agent must navigate, with brief
descriptions, and always show every path containing a **scoped-AGENTS.md** or `AGENTS.md` file, without adding a
description for those files. The root `AGENTS.md` must also state that a **scoped-AGENTS.md** is the root `AGENTS.md`
scoped to that path, and that agents must read every **scoped-AGENTS.md** on the access path.

Attach Skill triggers in `AGENTS.md` and **scoped-AGENTS.md** to concrete local actions or workflows. Each trigger
must add detail beyond the Skill's broad, high-compression frontmatter `description`, not restate it. Prefer
meaningful orchestration—use Skill A for a required pre-step, then Skill B to create the object—only when that
sequence is not already a generic workflow inside Skill A or Skill B.

## Skills

Repository-local Skill directories contain a `SKILL.md` with concise, discriminating frontmatter. Start every Skill
prompt by making clear that it is a guide, not a script. State the task boundary, the non-obvious local constraints,
and only the workflow detail that changes an Agent's decisions.

Name every Skill this scaffolding creates or maintains `<repository>-<skill>`, in lowercase hyphenated words, where
`repository` is the Git root directory name. Keep the frontmatter `name` equal to the directory name, so the prefix
stays unique and stable in this repository. This Skill follows its own rule and is maintained as
`research-scaffolding-builder`.

Add metadata, references, scripts, or assets only when they serve a concrete use. Cross-reference Skills, `AGENTS.md`,
and repository documents when that helps keep Agent behavior consistent; links may point to internal or external
authority.

When creating or adapting a Skill, keep its stated scope general enough for foreseeable repository evolution. Do not
narrow a rule to a current language, path, or artifact type unless the narrower boundary is intentional; if a wording
could hide other in-scope code or documents, generalize it. Keep a Skill free of another repository's names,
authorities, and file layout, so it stays usable here without that context.

Do not add a standalone "Related Skills" section merely to list related work. Reference another Skill at the concrete
step or check that requires it, and state what that Skill is used to do there.

## Runtime Context

When introducing or adapting a Skill, identify its real runtime assumptions: persistent work locations, path bases,
required tools, source documents, generated artifacts, or state that must survive a turn. Make those assumptions
usable in the surrounding scaffolding. The information may be conveyed naturally through repository layout, a scoped
`AGENTS.md`, linked context, or the Skill itself; choose the least intrusive form that keeps the Skill operational.

Do not add ceremonial instructions or duplicate repository data into a Skill when a concise reference to its
authority is sufficient.

## Review

Before handoff, verify that:

- the scope can be discovered from applicable Agent guidance;
- **scoped-AGENTS.md** guidance does not conflict;
- Skills this scaffolding owns, including any it renames, follow the `<repository>-<skill>` naming rule;
- Skills this scaffolding does not own are unchanged, and externally installed Skills were not renamed, rewritten, or
  deleted;
- cross-references resolve;
- every affected reference and link is valid;
- skill trigger timing text in `AGENTS.md` and **scoped-AGENTS.md** files has not regressed or retained outdated
  trigger descriptions;
- the result is complete without unnecessary process text;
- changed scaffolding prose and its surrounding context read consistently in the language fixed above.
