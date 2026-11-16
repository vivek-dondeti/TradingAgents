---
name: verify
description: Run the same checks CI runs (ruff, pytest, New York timezone pytest, clean-install import) to verify changes before committing or opening a PR.
---

Run these from the repo root, in order, and report the results of each step. Stop and fix the problem if a step fails.

1. `ruff check .` — must be clean. Do not run `ruff format`.
2. `pytest -q`
3. `TZ=America/New_York pytest -q` — catches tests that only pass in UTC.
4. Clean-install smoke check (catches undeclared runtime deps):
   ```
   python -m venv /tmp/ta-smoke && /tmp/ta-smoke/bin/pip install -q . && /tmp/ta-smoke/bin/python -c "import tradingagents, cli.main" ; rm -rf /tmp/ta-smoke
   ```
   Skip this step if only tests or docs changed.

If $ARGUMENTS names specific tests, run those first with `pytest -q $ARGUMENTS` before the full suite.
