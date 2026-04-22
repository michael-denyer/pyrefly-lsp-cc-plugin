# pyrefly-lsp-cc-plugin

A Claude Code plugin marketplace that ships [Pyrefly](https://pyrefly.org) —
Facebook's fast Python type checker and language server — as a drop-in LSP for
Claude Code.

## Plugins

| Plugin | Description |
|--------|-------------|
| [`pyrefly-lsp`](./plugins/pyrefly-lsp/) | Registers `pyrefly lsp` as the Python language server for `.py` / `.pyi` files. |

## Install

```text
/plugin marketplace add michael-denyer/pyrefly-lsp-cc-plugin
/plugin install pyrefly-lsp@pyrefly-lsp-cc-plugin
```

You must install the `pyrefly` binary yourself — see the plugin
[README](./plugins/pyrefly-lsp/README.md) for details.

## Design

See [docs/superpowers/specs/2026-04-22-pyrefly-lsp-cc-plugin-design.md](./docs/superpowers/specs/2026-04-22-pyrefly-lsp-cc-plugin-design.md)
for the full design rationale.

## Releases

Versioning is automated via
[Release Please](https://github.com/googleapis/release-please). Commits on
`main` follow the [Conventional Commits](https://www.conventionalcommits.org/)
spec:

- `feat: …` → minor bump
- `fix: …` → patch bump
- `feat!: …` or a `BREAKING CHANGE:` footer → major bump
- `chore: …`, `docs: …`, `ci: …` → no release

Release Please watches `main` and opens a *"chore: release x.y.z"* PR that
accumulates unreleased changes. Merging that PR tags the release, publishes
a GitHub Release, updates [`CHANGELOG.md`](./CHANGELOG.md), and propagates
the version into `plugins[0].version` in
[`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json).

## License

MIT. See [LICENSE](./LICENSE).
