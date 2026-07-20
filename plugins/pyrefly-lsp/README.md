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
3. Otherwise launches `pyrefly lsp` and watches its memory (see below).

Pyrefly speaks LSP on stdio by default — there is no `--stdio` flag to pass.

Handles: `.py`, `.pyi`.

## Memory cap

Pyrefly has known unbounded-memory-growth issues on long-lived servers
([facebook/pyrefly#1092](https://github.com/facebook/pyrefly/issues/1092),
[#2302](https://github.com/facebook/pyrefly/issues/2302),
[#2970](https://github.com/facebook/pyrefly/issues/2970)) — a server left
running for days can climb from a few hundred MB to multiple GB. The shim
polls the server's RSS every 30 seconds and kills it when it crosses
**750 MiB** (default). Claude Code respawns the server and re-initializes on
the next LSP request, so the only cost is a fresh index.

The shim cannot restart pyrefly itself — the LSP client owns the
`initialize` handshake — which is why it exits and lets the client respawn.

Tune via environment variables:

| Variable | Default | Meaning |
|---|---|---|
| `PYREFLY_LSP_MAX_RSS_KB` | `768000` (750 MiB) | RSS cap in KB; `0` disables the cap entirely |
| `PYREFLY_LSP_POLL_SECS` | `30` | Seconds between RSS checks |

When the cap fires, the shim logs one line to stderr
(`pyrefly-lsp: RSS ...KB exceeded cap ...KB`), visible in Claude Code's LSP
logs.

## Configuring Pyrefly (optional)

Pyrefly auto-discovers config from your project root. It reads **one of**:

- `pyrefly.toml` at the project root (note: no leading dot — `.pyrefly.toml`
  is silently ignored), or
- a `[tool.pyrefly]` table inside `pyproject.toml`

### Starter snippet

```toml
[tool.pyrefly]
project-includes = ["src", "tests"]
python-version = "3.12"

# Pyrefly has no `strict = true` flag — strictness is opt-in per error code.
# Start at "warn" so CI stays green during adoption, then flip to the
# default severity ("error") once the codebase is clean.
[tool.pyrefly.errors]
implicit-any = "warn"
implicitly-defined-attribute = "warn"
missing-override-decorator = "warn"
unannotated-parameter = "warn"
unannotated-return = "warn"
untyped-import = "warn"
unused-ignore = "warn"

# Relax the noisy checks in tests.
[[tool.pyrefly.sub-config]]
matches = "tests/**"
[tool.pyrefly.sub-config.errors]
implicit-any = false
unannotated-parameter = false
unannotated-return = false
```

### Gotchas

- **No `strict = true` equivalent** — enumerate codes in `[tool.pyrefly.errors]`.
- **`project-includes` fails hard on zero matches** — delete a directory and
  forget to update the glob, and Pyrefly refuses to start.
- **Adopting on an existing codebase:** generate a baseline first with
  `pyrefly check --baseline=pyrefly-baseline.json --update-baseline`, then
  add `baseline = "pyrefly-baseline.json"` to the config. Otherwise every
  PR gets blocked on pre-existing errors.

Full option reference:
[facebook/pyrefly configuration docs](https://pyrefly.org/en/docs/configuration/).

## License

MIT. See [LICENSE](./LICENSE).
