# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup and commands

- Dev install: `pip install -e ".[dev]"` (Python >=3.11; CI covers 3.11–3.14). Optional extra `.[bedrock]`.
- Lint: `ruff check .` — must stay fully clean (CI enforces it). Do NOT run `ruff format` across the repo; it is intentionally not applied yet to avoid conflicts with open PRs.
- Tests: `pytest -q` (addopts exclude `integration`; run those with `-m integration`). Markers are strict — register new ones in `pyproject.toml`.
- CI also runs the suite under `TZ=America/New_York`, so avoid tests that only pass in UTC.
- Runtime deps must be declared in `pyproject.toml`: CI does a bare `pip install .` and imports `tradingagents` and `cli.main`.
- CLI: `tradingagents` (Typer, `cli/main.py`); headless: `tradingagents analyze --ticker NVDA --date 2026-09-01 ...`.

## Test harness quirks (`tests/conftest.py`)

- Network is blocked (sockets, DNS, curl_cffi) except in `integration` tests — mock vendor calls instead.
- All `TRADINGAGENTS_*` env vars are blanked before import, results/cache/memory paths point to tmp, and provider API keys are set to `"placeholder"`.
- `dataflows.config._config` is reset to DEFAULT_CONFIG per test because `set_config` merges, never clears.
- `tests/test_layering.py` fails if `yfinance` is imported outside `tradingagents/dataflows/`.

## Architecture rules

- Config defaults: `tradingagents/default_config.py`. To make a key env-settable, add a row to `_ENV_OVERRIDES` (values coerce to the default's type; invalid values raise). Importing `tradingagents` loads `.env` and freezes `DEFAULT_CONFIG`.
- Data access goes through `dataflows/router.py` (`route_to_vendor`). The configured vendor chain is exact — never add silent fallback to a vendor the user didn't select. `tool_vendors` overrides `data_vendors`. Only `macro_data`/`prediction_markets` may degrade to a sentinel; core categories raise typed errors (`dataflows/errors.py`).
- Look-ahead bias is a correctness bug: any data for a run dated *d* must be what was knowable as of *d* (news, filings, social posts, fundamentals). Preserve the existing as-of guards and add tests for new data paths.
- LLM providers: `tradingagents/llm_clients/factory.py`; non-native providers are OpenAI-compatible entries in the registry in `openai_client.py`/`validators.py`. `backend_url` defaults to None on purpose so one provider's URL doesn't leak into another.
- Quick-think tier = analysts, researchers, debaters, trader; deep-think tier = managers.

## Repo etiquette

- Commits use Conventional Commits: `type(scope): subject` (e.g. `fix(llm): ...`).
- Do not edit `CHANGELOG.md` — maintainers write entries.
