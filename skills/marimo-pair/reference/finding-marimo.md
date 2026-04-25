# Finding and Invoking marimo

**Project-specific override:** this repo (`/home/broniy/datascience`) is
uv-managed, with `marimo` pinned in `pyproject.toml` and installed in the
repo-root `.venv/`. In this project you MUST always invoke marimo as:

```sh
uv run marimo edit <notebook>.py --no-token
```

Run it from the repo root so uv picks up the project's `pyproject.toml` and
`.venv/`. Do NOT use `--sandbox`, `uvx`, `uv run --with ...`, `pipx`, or a
globally installed marimo — the agent and the user must share the same
environment.

Only servers started with `--no-token` register in the local server registry
and are auto-discoverable. If a server has a token, set the `MARIMO_TOKEN`
environment variable before calling the execute script (avoids leaking the
token in process listings).

## Verifying marimo is actually in the env

Before starting a server, you can sanity-check with:

```sh
uv run python -c "import marimo; print(marimo.__version__)"
```

If this fails, the fix is `uv sync` at the repo root — NOT switching to
`--sandbox` or `uvx`. Stop and tell the user if `uv sync` doesn't resolve it.

## Flags

- `--no-token` — required for auto-discovery by the skill's scripts.
- `--headless` — DO NOT pass unless the user explicitly asks. Default
  behavior (auto-opening the browser) is the expected pairing experience.
- `--sandbox` — FORBIDDEN in this project.

## Version requirement

`code_mode` shipped in marimo v0.21.1. If the project's pinned version is
older, ask the user to bump marimo in `pyproject.toml` and run `uv sync`
before proceeding.
