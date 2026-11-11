# deepseek-harness

## Scope

DeepSeek Harness (`dsh`), the open-source agent harness published by DeepSeek AI, studied at release tag
`dsh-v0.2.1-rc.3`, and Cordis, the vendored plugin framework it is built on (`vendor/cordis/`).

## Status

active

## Questions

- How is Cordis designed: which abstractions does it introduce, why, and which TypeScript features carry them?

## Documents

- [cordis-design.md](cordis-design.md) — framework level: runtime objects, the two loops that drive them, contracts
  between abstractions, invariants, trade-offs. Read this first.
- [cordis-model.md](cordis-model.md) — programming model: the conventions a plugin author must follow, why they exist, and what breaks without them.
- [cordis-context.md](cordis-context.md) — Context: proxy container, scope chain, service resolution, shadow/receiver.
- [cordis-service.md](cordis-service.md) — Service: declaration, lifetime, availability predicate, intercept config.
- [cordis-fiber.md](cordis-fiber.md) — Fiber: state machine, dependency epoch, effect protocol, teardown order.
- [cordis-events.md](cordis-events.md) — Events: dispatch modes, listener lifetime, the `internal/*` surface.
- [cordis-registry.md](cordis-registry.md) — Registry: plugin shapes, runtime vs fiber, config validation.
- [cordis-traceable.md](cordis-traceable.md) — ownership model: what traceable / shadow / noShadow / receiver actually do.

## Revision Note

The research documents were originally written against `dsh-v0.1.5-rc.2` (commit
`fb2c4b9e698e30edb738bca4cf0618587db7d203`). The source was upgraded to `dsh-v0.2.1-rc.3` (commit
`1f81c047fc48fa190a683e469daa699857e9afe8`) on 2026-10-26; the main text now follows the new tag, line citations were
updated, and behaviour that differs from the original revision is recorded in the inline historical notes. Cordis core changed only in `events.ts`, `fiber.ts`, `index.ts`, and
`logger.ts` (plus `vendor/cordis/README.md` and `package.json`); `context.ts`, `reflect.ts`, `service.ts`,
`registry.ts`, and `utils.ts` are byte-identical between the two tags.

Notable semantic changes in the new tag: `Fiber.update()` no longer returns the `internal/update` waterfall result and
its `next` continuation is typed as `() => void`; `LoggerService.exporter()` disposal now keeps a per-registration id;
the loader gained volatile-config commit paths (`loader/volatile-update`) that apply volatile-only changes without a
remount. The Cordis fiber lifecycle is otherwise unchanged.

Inline **Changed in `dsh-…`** notes are historical reference notes: the surrounding prose describes the current pinned
revision, and each note records the behaviour of the historical reference revision. Historical citations link to the separate Ref
`sources/deepseek-harness-v0.1.5-rc.2`, so both revisions can be read side by side.

## Source Index

- Git repository `https://github.com/deepseek-ai/deepseek-harness`, release tag `dsh-v0.2.1-rc.3`, commit
  `1f81c047fc48fa190a683e469daa699857e9afe8`; imported 2026-09-14, upgraded and re-checked 2026-10-26.
- Pinned read-only copy: [sources/deepseek-harness](sources/deepseek-harness)<!-- doctidex: {cross-boundary-point: sources/deepseek-harness} -->
- Historical reference revision: release tag `dsh-v0.1.5-rc.2`, commit `fb2c4b9e698e30edb738bca4cf0618587db7d203`; read 2026-09-17. Pinned copy: [sources/deepseek-harness-v0.1.5-rc.2](sources/deepseek-harness-v0.1.5-rc.2)<!-- doctidex: {cross-boundary-point: sources/deepseek-harness-v0.1.5-rc.2} -->
- Paper, not installable, cited by URL: Yifan Shi, Wei Zhang, Tianyi Cui, "A Programming Paradigm for Spatiotemporal
  Composability", [arXiv:2608.25512](https://arxiv.org/abs/2608.25512), submitted 2026-08-26; abstract read 2026-09-17.
