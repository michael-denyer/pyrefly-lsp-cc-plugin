# pyrefly-lsp

Wires [Pyrefly](https://pyrefly.org) — Facebook's fast Python type checker and
language server — into Claude Code's built-in LSP integration.

Intended as a drop-in faster alternative to the upstream `pyright-lsp` plugin.

## Prerequisites

Install the `pyrefly` binary on your host. The plugin does **not** bundle it.

```bash
uv tool install pyrefly
# or: pipx install pyrefly
# or: pip install pyrefly
```

Verify it is on your `PATH`:

```bash
pyrefly --version
```

## Install

From inside Claude Code:

```text
/plugin marketplace add michael-denyer/pyrefly-lsp-cc-plugin
/plugin install pyrefly-lsp@pyrefly-lsp-cc-plugin
```

Once installed, Claude Code will launch `pyrefly lsp` on stdio for any `.py` or
`.pyi` file you open.

## How it works

The plugin is a single marketplace entry that registers an LSP server:

- `command`: `pyrefly`
- `args`: `["lsp"]`
- Handles: `.py`, `.pyi`

Pyrefly speaks LSP on stdio by default — there is no `--stdio` flag to pass.

## License

MIT. See [LICENSE](./LICENSE).
