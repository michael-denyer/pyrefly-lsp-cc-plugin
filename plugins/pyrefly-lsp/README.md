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

The plugin registers an LSP server that points at a small shim script
bundled with the plugin ([`bin/pyrefly-lsp`](./bin/pyrefly-lsp)). The shim:

1. Checks whether `pyrefly` is on `PATH`.
2. If missing, exits with a clear stderr message listing install commands —
   so Claude Code's LSP error surface tells you the fix rather than
   "command not found".
3. Otherwise `exec`s `pyrefly lsp`.

Pyrefly speaks LSP on stdio by default — there is no `--stdio` flag to pass.

Handles: `.py`, `.pyi`.

## License

MIT. See [LICENSE](./LICENSE).
