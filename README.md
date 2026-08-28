# Morning Dashboard

A personal "AI operating system" dashboard: every morning at 7:00 AM, a Python
script running on my Mac pulls today's calendar, unread email, tasks, weather,
and news headlines, asks Claude to write a short morning brief, and opens a
single `dashboard.html` page in the browser.

Everything runs locally and every service used is free.

## How it works

```
launchd (7:00 AM)
   └─> python -m dashboard.main
         ├─ fetch: Google Calendar + Google Tasks (OAuth, read-only)
         ├─ fetch: Gmail unread, last 24h (IMAP + app password)
         ├─ fetch: weather (Open-Meteo, no key) + headlines (RSS)
         ├─ save:  data/YYYY-MM-DD.json snapshot
         ├─ brief: `claude -p` writes the morning summary
         └─ render: Jinja2 -> output/dashboard.html -> opens in browser
```

## Architecture (bottom-up)

```
+---+----------------+----------------------------+----------------------------------+
| # | Layer          | Component                  | Responsibility                   |
+---+----------------+----------------------------+----------------------------------+
| 1 | Types/models   | dashboard/models.py        | dataclasses for all data         |
| 2 | Units          | dashboard/fetchers/*.py    | one fetcher per source           |
| 2 | Units          | dashboard/brief.py         | AI morning brief via claude CLI  |
| 2 | Units          | dashboard/render.py        | Jinja2 template -> HTML          |
| 3 | Composition    | dashboard/main.py          | orchestrates fetch->brief->render|
| 4 | Entry point    | launchd plist + run.sh     | daily 7:00 AM automation         |
+---+----------------+----------------------------+----------------------------------+
```

Every fetcher is split into **fetch** (network call) and **parse** (pure function,
raw response -> dataclasses). Parse functions are unit-tested against fixture
files in `tests/fixtures/` — tests never touch the network.

## Project status

- [x] M0 — repo scaffold, models, CI, templates
- [ ] M1 — weather + news fetchers (no auth needed)
- [ ] M2 — HTML rendering + `--demo` mode
- [ ] M3 — Google Calendar + Tasks (OAuth)
- [ ] M4 — Gmail via IMAP
- [ ] M5 — AI morning brief
- [ ] M6 — launchd automation

## Local setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

# verify everything works
ruff check . && ruff format --check . && pytest
```

Secrets (`secrets/`, `.env`) are gitignored and documented in `.env.example`
as each milestone introduces them.

## Development workflow

Every change follows the same loop — see [docs/workflow.md](docs/workflow.md):
issue → branch → small conventional commits → PR → green CI → review → squash-merge.
`main` is protected; nothing lands without a pull request and passing checks.
