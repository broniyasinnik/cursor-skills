---
name: marimo-pair
description: >-
  Work inside a running marimo notebook's kernel — execute code, create cells,
  and build a notebook as an artifact. Use when the user wants to start a
  marimo notebook or work in an active marimo session.
allowed-tools: Bash(bash **/scripts/discover-servers.sh *), Bash(bash **/scripts/execute-code.sh *), Read
---

# marimo Pair Programming Protocol

This skill gives you full access to a running marimo notebook. You can read
cell code, create and edit cells, install packages, run cells, and inspect
the reactive graph — all programmatically. The user sees results live in their
browser while you work through bundled scripts or MCP.

## Philosophy

marimo notebooks are a dataflow graph — cells are the fundamental unit of
computation, connected by the variables they define and reference. When a cell
runs, marimo automatically re-executes downstream cells. You have full access
to the running notebook.

- **Cells are your main lever.** Use them to break up work and choose how and
  when to bring the human into the loop. Not every cell needs rich output —
  sometimes the object itself is enough, sometimes a summary is better.
  Match the presentation to the intent.
- **Understand intent first.** When clear, act. When ambiguous, clarify.
- **Follow existing signal.** Check imports, `pyproject.toml`, existing cells,
  and `dir(ctx)` before reaching for external tools.
- **Stay focused.** Build first, polish later — cell names, layout, and styling
  can wait.

## Prerequisites

### How to invoke marimo

**This project is uv-managed and already has `marimo` in `pyproject.toml` and
a populated `.venv/` at the repo root. ALWAYS start marimo via the project
environment — NEVER use `--sandbox`, `uvx`, `uv run --with ...`, or any other
isolated/ephemeral runner. The whole point is that the agent shares the same
environment the user already has.**

Use exactly this form, run from the repo root (`/home/broniy/datascience`):

```sh
uv run marimo edit <notebook>.py --no-token
```

Do NOT use any of these in this project:

- `uvx marimo@latest edit ... --sandbox`
- `marimo edit ... --sandbox`
- `uv run --with marimo marimo edit ...`
- manually activating `.venv` and invoking `marimo` directly

If `uv run marimo` ever fails because marimo is missing from the env, stop
and tell the user — do not silently fall back to sandbox mode. The fix is
`uv sync` at the repo root, not a new isolated environment.

Only servers started with `--no-token` register in the local server registry
and are auto-discoverable — starting without a token makes discovery easier.
If a server has a token, set the `MARIMO_TOKEN` environment variable before
calling the execute script (avoids leaking the token in process listings).

**`--headless` guidance depends on the host:**

- **Local desktop** (agent is running on the same machine as the user's browser):
  omit `--headless`. marimo auto-opens the browser, which is the expected pairing experience.
- **Remote host / SSH / Cursor remote / GCP VM / container with no display**
  (detectable via `$SSH_CONNECTION`, `$DISPLAY` unset, or the hostname having no GUI):
  pass `--headless`, print the URL prominently, and ask the user to open it
  in their browser. The auto-open will fail on these hosts anyway and just
  spams the log. After the user opens the URL, a kernel session is created
  and `execute-code.sh` can attach.

If the user explicitly requests headless on a local machine, offer to open it
with `open http://localhost:<port>`.

See [finding-marimo.md](reference/finding-marimo.md) only for background on
auto-discovery mechanics — the invocation rule above overrides any decision
tree there.

### Running inside Cursor's shell sandbox

If you're a Cursor agent: **every call that touches marimo must run outside
the default Cursor shell sandbox.** That means passing
`required_permissions: ["all"]` on the Shell tool when:

- Starting the marimo server (`uv run marimo edit ...`)
- Running `scripts/discover-servers.sh`
- Running `scripts/execute-code.sh`

The sandbox blocks writes to `~/.local/state/marimo/` and prevents the
listening port from being reachable on the real loopback interface. The
symptoms are subtle because marimo still *prints* a URL and exits 0:

- marimo logs `PermissionError: [Errno 13] Permission denied: '/home/.../state/marimo/state.toml'`
- marimo logs `Failed to register server: [Errno 13] Permission denied: '.../servers/tmp*.tmp'`
- `discover-servers.sh` returns `[]` even though a server is "running"
- `curl http://localhost:<port>` returns `000` / `ss -ltn` shows no listener
- `discover-servers.sh` prints `rm: cannot remove '.../127.0.0.1_<port>.json': Permission denied` for stale entries

If you see any of these, **kill the sandboxed marimo process and restart it
with `required_permissions: ["all"]`**. Don't waste time debugging the port
— the sandbox is the root cause.

Rule of thumb: from a Cursor agent, always launch marimo as a background
Shell with `block_until_ms: 0` AND `required_permissions: ["all"]`, and use
`--url http://localhost:<port>` on `execute-code.sh` (also with `all`
permissions) since the registry may still be stale/unreadable from past
sandboxed runs.

## Troubleshooting

### `execute-code.sh` says "No active sessions on the server"

The server is up but no notebook is open in a browser yet — `code_mode`
needs an active kernel session, and a session is only created when a user
loads the notebook URL in a browser tab. This is **not** a connection
failure and does **not** mean you need to restart the server.

Fix: ask the user to open `http://localhost:<port>` (or the path to the
specific notebook) in their browser, then retry. On a fresh server this is
expected on the very first `execute-code` call of the session.

### User keeps getting prompted to allow Bash commands

The skill declares `allowed-tools` in its frontmatter, but Claude Code may
still prompt for each Bash call. To fix this, the user should add the absolute
paths to the scripts to their `.claude/settings.json` (project-level) or
`~/.claude/settings.json` (global):

```json
{
  "permissions": {
    "allow": [
      "Bash(bash /absolute/path/to/skills/marimo-pair/scripts/discover-servers.sh *)",
      "Bash(bash /absolute/path/to/skills/marimo-pair/scripts/execute-code.sh *)"
    ]
  }
}
```

## How to Discover Servers and Execute Code

Two operations: **discover servers** and **execute code**.

| Operation | Script | MCP |
|-----------|--------|-----|
| Discover servers | `bash scripts/discover-servers.sh` | `list_sessions()` tool |
| Execute code | `bash scripts/execute-code.sh -c "code"` | `execute_code(code=..., session_id=...)` tool |
| Execute code (multiline) | `bash scripts/execute-code.sh <<'EOF'` | same |
| Execute code (direct URL) | `bash scripts/execute-code.sh --url URL -c "code"` | same (with `url` param) |

Scripts auto-discover sessions from the registry on disk. Use `--port` to
target a specific server when multiple are running, `--session` to target a
specific session when multiple notebooks are open on the same server, or
`--url` to skip discovery entirely and hit a server URL directly (e.g.
`--url http://localhost:2718`). `--url` is the only way to connect to
remote servers since auto-discovery only reads the local registry. **Only
use `--url` with trusted servers** — data is sent to the endpoint, so a
malicious URL could exfiltrate notebook contents. Set the `MARIMO_TOKEN`
env var to authenticate when the server has token auth enabled (`--token`
flag also works but exposes the token in process listings). If the
server was started with `--mcp`, you'll have MCP tools available as an
alternative.

### Discovery finds nothing but the user has a server running?

Only `--no-token` servers are in the registry. If discovery comes up empty,
the server likely has token auth — ask the user for the token and set it as
the `MARIMO_TOKEN` environment variable.

### No servers running?

**Always discover before starting.** Background task "completed" notifications
do not mean the server died — check the output or run discover first.

If no servers are found, read the user's intent — if they want a notebook,
start one. **Always start marimo as a background task** (using
`run_in_background` on the Bash tool) so the server automatically gets cleaned
up when the session ends and doesn't block the conversation. See
[finding-marimo.md](reference/finding-marimo.md).

If there's no `.py` file yet, pick a descriptive filename based on context
(e.g., `exploration.py`, `analysis.py`, `dashboard.py`). Don't ask — just
pick something reasonable.

**Avoid shell escaping issues.** `-c` works for simple one-liners, but for
multiline code or code with quotes/backticks/`${}`, use a heredoc or a file:

```bash
# heredoc (single-quoted delimiter prevents shell interpolation)
bash scripts/execute-code.sh <<'EOF'
import marimo._code_mode as cm

async with cm.get_context() as ctx:
    ctx.create_cell("x = 1")
EOF

# file
bash scripts/execute-code.sh /tmp/code.py

# direct URL (skips auto-discovery and works with remote servers)
bash scripts/execute-code.sh --url http://localhost:2718 -c "1 + 1"
```

## Executing Code

Every execute-code call runs inside the notebook's kernel. All cell variables
are in scope — `print(df.head())` just works. Nothing you define persists
between calls (variables, imports, side-effects all reset), but you can freely
introspect the notebook: inspect variables, test code snippets, check types
and shapes. Use this to explore, prototype, and validate before committing
anything to the notebook — then create cells to persist state and make results
visible to the user.

To mutate the notebook's dataflow graph — create, edit, and delete cells,
install packages, and run cells — use `marimo._code_mode`:

```python
import marimo._code_mode as cm

async with cm.get_context() as ctx:
    cid = ctx.create_cell("x = 1")
    ctx.install_packages("pandas")
    ctx.run_cell(cid)
```

You **must** use `async with` — without it, operations silently do nothing.
All `ctx.*` methods are **synchronous** — they queue operations and the
context manager flushes them on exit. Do **not** `await` them.

**Cells are not auto-executed.** `create_cell` and `edit_cell` are structural
changes only — use `run_cell` to queue execution.

`code_mode` is a tested, safe API for notebook mutations — prefer it for all
structural changes. You also have access to marimo internals from the kernel,
but treat that as a last resort and only with high confidence after exploration.

**UI state lives outside the reactive graph.** Anywidget traitlets can be read
or set directly (e.g., `slider.value = 5`). For `mo.ui.*` elements, use
`ctx.set_ui_value(element, new_value)` inside `code_mode`.

### First Step: Explore the API

The `code_mode` API can change between marimo versions — and each running
server could be a different version. Inspect what's available at the start of
each session, especially when switching between servers.

```python
import marimo._code_mode as cm

async with cm.get_context() as ctx:
    ctx  # inspect me — dir(), help(), .cells, ...
```

## Guard Rails

Skip these and the UI breaks:

- **Install packages via `ctx.install_packages()`, not `uv add` or `pip`.**
  The code API handles kernel restarts and dependency resolution correctly.
  Only fall back to external CLIs if the API is unavailable or fails.
- **Custom widget = anywidget.** For bespoke visual components, use anywidget
  with HTML/CSS/JS. Composed `mo.ui` is fine for simple forms and controls.
  See [rich-representations.md](reference/rich-representations.md).
- **NEVER write to the `.py` file directly while a session is running — the kernel owns it.**
- **No temp-file deps in cells.** `pathlib.Path("/tmp/...")` in cell code is a bug.
- **Avoid empty cells.** Prefer `edit_cell` into existing empty cells rather
  than creating new ones. Clean up any cells that end up empty after edits.
- **Don't worry about cell names.** Most cells don't need explicit names —
  see [notebook-improvements.md](reference/notebook-improvements.md#cell-names).

## Widgets and Reactivity

Anywidget state (traitlets) lives outside marimo's reactive graph. To hook a
widget trait into the graph, pick one strategy per widget — never mix them:

- **`mo.state` + `.observe()`** — you pick specific traits to bridge. Default choice.
- **`mo.ui.anywidget()`** — wraps all synced traits into one reactive `.value`. Convenient but coarser.

Read [rich-representations.md](reference/rich-representations.md) before wiring either.

## Keep in Mind

- **The user is editing too.** The notebook can change between your calls —
  re-inspect notebook state if it's been a while since you last looked.
- **Deletions are destructive.** Deleting a cell removes its variables from
  kernel memory — restoring means recreating the cell and re-running it and
  its dependents. If intent seems ambiguous, ask first.
- **Installing packages changes the project.** `ctx.install_packages()` adds
  real dependencies — confirm when it's not obvious from context.

## References

- [finding-marimo.md](reference/finding-marimo.md) — how to find and invoke the right marimo
- [gotchas.md](reference/gotchas.md) — cached module proxies and other traps
- [rich-representations.md](reference/rich-representations.md) — custom widgets and visualizations
- [notebook-improvements.md](reference/notebook-improvements.md) — improving existing notebooks
