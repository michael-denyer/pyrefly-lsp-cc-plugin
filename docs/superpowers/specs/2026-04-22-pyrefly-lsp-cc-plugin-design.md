# pyrefly-lsp-cc-plugin design

**Date:** 2026-04-22
**Status:** approved

## Goal

Ship a Claude Code plugin that wires [Pyrefly](https://pyrefly.org) (Facebook's
fast Python type checker and language server) into Claude Code's built-in LSP
integration. Intended as a drop-in faster alternative to the upstream
`pyright-lsp` plugin.

## Non-goals

- Bundling the Pyrefly binary. Users install `pyrefly` themselves via pip / uv /
  pipx.
- Adding any skills, agents, commands, or hooks. This is a pure LSP-wiring
  plugin.
- Automated tests or CI. The plugin is a JSON manifest plus a README; there is
  nothing executable to test in this repo.

## Repository layout

```text
pyrefly-lsp-cc-plugin/
├── .claude-plugin/
│   └── marketplace.json              # marketplace manifest (LSP config lives here)
├── plugins/
│   └── pyrefly-lsp/
│       ├── README.md                 # install instructions for end users
│       └── LICENSE                   # MIT
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-04-22-pyrefly-lsp-cc-plugin-design.md
├── README.md                         # top-level: what this marketplace is
└── LICENSE                           # MIT (covers the repo as a whole)
```

A marketplace-style layout was chosen over a bare single-plugin layout so
additional plugins (e.g. a future `ruff-lsp` wrapper) can be added without
restructuring. The cost is one extra JSON file today; the benefit is no
retrofit later.

## Marketplace manifest

`.claude-plugin/marketplace.json`:

```json
{
  "name": "pyrefly-lsp-cc-plugin",
  "owner": { "name": "Michael Denyer" },
  "plugins": [
    {
      "name": "pyrefly-lsp",
      "description": "Python language server (Pyrefly) — fast type checking and code intelligence",
      "version": "0.1.0",
      "author": { "name": "Michael Denyer" },
      "source": "./plugins/pyrefly-lsp",
      "category": "development",
      "strict": false,
      "lspServers": {
        "pyrefly": {
          "command": "pyrefly",
          "args": ["lsp"],
          "extensionToLanguage": {
            ".py": "python",
            ".pyi": "python"
          }
        }
      }
    }
  ]
}
```

Key differences from the `pyright-lsp` entry in `claude-plugins-official`:

- `command` is `pyrefly` (not `pyright-langserver`)
- `args` is `["lsp"]` (Pyrefly uses an `lsp` subcommand; it speaks LSP on stdio
  by default and does not take a `--stdio` flag)
- Everything else (extension mapping, category, strict flag) mirrors the
  upstream pyright entry

## User-facing install flow

1. Install Pyrefly on the host:
   ```bash
   uv tool install pyrefly   # or pipx install pyrefly, or pip install pyrefly
   ```
2. Add the marketplace in Claude Code:
   ```text
   /plugin marketplace add michael-denyer/pyrefly-lsp-cc-plugin
   ```
3. Install the plugin:
   ```text
   /plugin install pyrefly-lsp@pyrefly-lsp-cc-plugin
   ```

## Licensing

MIT, at both the repo root and the plugin directory. Rationale:

- Permissive licensing matches the broader Claude Code plugin ecosystem.
- The plugin content is a JSON manifest and a README — there is nothing
  substantive to copyleft-protect.
- MIT is one paragraph and imposes the least friction on anyone who wants to
  fork, mirror, or extend the plugin.

## What we are not doing (YAGNI)

- No CI workflow. Nothing here builds or tests.
- No version-bump automation. Version bumps are a one-line edit to
  `marketplace.json`.
- No pre-commit hooks. No source code to lint.
- No bundled `pyrefly` binary. Distribution is the user's responsibility.
