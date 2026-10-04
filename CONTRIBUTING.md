# Contributing to TACLEX ML

Thank you for considering contributing to TACLEX ML. This document tells
you everything you need to know to make your first contribution.

TACLEX is a multi-game mission generation platform. Since v2.4, the codebase
is split into 3 layers — please read [Architecture](#architecture) before
contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Ways to Contribute](#ways-to-contribute)
- [Architecture](#architecture)
- [Development Setup](#development-setup)
- [Adding a New Game Adapter](#adding-a-new-game-adapter)
- [Adding a New Content Pack](#adding-a-new-content-pack)
- [Adding a New Trigger or Objective](#adding-a-new-trigger-or-objective)
- [Running Tests](#running-tests)
- [Code Style](#code-style)
- [Submitting Changes](#submitting-changes)
- [Reporting Bugs](#reporting-bugs)
- [Requesting Features](#requesting-features)

## Code of Conduct

Be respectful. This is an open-source project maintained by volunteers.
Zero tolerance for harassment, discrimination, or personal attacks.

## Ways to Contribute

You don't need to write code to help:

- **Report bugs** — open an issue with reproduction steps
- **Suggest features** — open an issue with use case
- **Improve docs** — README, CHANGELOG, comments
- **Test in-game** — try missions, report anomalies
- **Translate prompts** — the LLM supports multiple languages
- **Create content packs** — for RHS, CUP, or other mods
- **Share screenshots** — help others see what's possible

## Architecture

Since v2.4, the project has 3 layers:

    TACLEX_CORE/       game-agnostic engine (11 modules)
    ADAPTERS/          one per game (arma3, dayz, reforger)
    CONTENT_PACKS/     JSON data per game
    PROMPTS/           LLM prompt per game
    GAMES/             build artifacts (SQF, PBO)

**Golden rule:** the CORE must not know about any specific game.

If you're adding Arma 3 logic, it goes in `ADAPTERS/arma3/`.
If you're adding DayZ data, it goes in `CONTENT_PACKS/dayz.json`.

If you're tempted to write `if game == "arma3"` inside `TACLEX_CORE/`,
stop. That's a leak. Open an issue instead.

See `TACLEX_CORE/README.md` for module-level details.

## Development Setup

### Requirements

- Python 3.12+
- Git 2.40+
- Windows 10/11 (Linux/macOS untested for the SQF side)
- Arma 3 (only for in-game tests)
- HEMTT (only for SQF rebuilds)

### Clone & Install

    git clone https://github.com/TreeoM-ManhheiM/TAC-LEX.git
    cd TAC-LEX
    pip install -r requirements.txt

### Environment

Copy `.env.example` to `taclax.env` and add your API key:

    LLM_PROVIDER=groq
    LLM_MODEL=openai/gpt-oss-120b
    LLM_API_KEY_1=your_key_here

Free Groq key: https://console.groq.com

### Run the Backend

    python taclex_core_standalone.py

Server will listen on http://127.0.0.1:8000

### Test Without Arma 3

    python core_test.py
    python TACLEX_CORE/campaign_test.py

Both should pass 100%. This is the fastest feedback loop.

## Adding a New Game Adapter

To add support for a new game (e.g., Arma 2, Starfield, etc.):

1. Create `ADAPTERS/<game>/__init__.py`
2. Create `ADAPTERS/<game>/renderer.py` with a `render(data)` function
3. The function receives a mission dict and returns `{"action": str, "args": list}`
4. Add a content pack: `CONTENT_PACKS/<game>.json`
5. Add a prompt pack: `PROMPTS/<game>.json`
6. Add a test in `core_test.py::test_multi_game`

**Example skeleton:**

    def render(data):
        intent = data.get("intent", "custom_mission")
        if intent == "custom_mission":
            return {
                "action": "<game>_mission",
                "args": [data.get("mission_name", ""), ...]
            }
        return {"action": "unknown", "args": []}

Reference implementations:
- `ADAPTERS/arma3/renderer.py` — PIPE format
- `ADAPTERS/dayz/renderer.py` — PIPE format
- `ADAPTERS/reforger/renderer.py` — JSON format

**Do not modify TACLEX_CORE to add a new adapter.** If you need to,
that's a design bug. Open an issue.

## Adding a New Content Pack

Content packs define vehicles, classes, triggers, and campaign rules
for a specific mod or game.

1. Copy `CONTENT_PACKS/arma3_vanilla.json` as a starting point
2. Edit to match your mod/game
3. Set `TACLEX_CONTENT_PACK=<your_pack>` in `taclax.env`
4. Test with `python core_test.py`

See `CONTENT_PACKS/README.md` for schema.

## Adding a New Trigger or Objective

**Triggers** live in `CONTENT_PACKS/*.json` under `triggers`. No Python
or SQF change required — the rules engine reads them dynamically.

    "my_trigger": {
      "keywords": ["my keyword", "outra palavra"],
      "condition": "time_elapsed",
      "condition_value": 60,
      "action": "spawn_group",
      "action_value": "",
      "count": 4
    }

**Objectives** are harder because they have SQF logic in the runtime.
Open an issue first to discuss design before submitting a PR.

## Running Tests

    # Core decoupling (must pass 8/8)
    python core_test.py

    # Campaign engine (must pass 10/10)
    python TACLEX_CORE/campaign_test.py

    # HTTP smoke test (requires backend running)
    python tests/http_smoke.py

All tests must pass before submitting a PR. If you add a feature,
add a test for it.

## Code Style

Python:
- 4 spaces indentation
- `snake_case` for functions and variables
- `PascalCase` for classes
- Double quotes for strings
- Comments in Portuguese (existing convention) or English
- Max line length: 100 chars

SQF:
- 4 spaces indentation
- `_privateVariable` prefix for private vars
- `TACLEX_fnc_` prefix for functions
- Log with `diag_log format ["[TACLEX_*] %1", ...]`

JSON:
- 2 spaces indentation
- `_schema`, `_pack_id`, `_version` fields required in content packs

## Submitting Changes

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make changes
4. Run tests: `python core_test.py && python TACLEX_CORE/campaign_test.py`
5. Commit: `git commit -m "Add: short description"`
6. Push: `git push origin feature/my-feature`
7. Open a Pull Request

**PR requirements:**
- Description explains WHY, not just WHAT
- Tests pass
- No modification to `TACLEX_CORE/` unless discussed in an issue
- Follows code style

**Commit message format:**

    Add: new feature
    Fix: bug description
    Docs: documentation change
    Refactor: internal change
    Test: test change

## Reporting Bugs

Open an issue with:

- **What you did** (input text, mods loaded)
- **What you expected**
- **What happened** (screenshot, RPT log if relevant)
- **Backend log** (last 20 lines from TACLEX_Control.exe)
- **Versions**: TACLEX version, Arma 3 version, mods

RPT logs are at:
    Documents\Arma 3\TACLEX_dataset\missions.jsonl

## Requesting Features

Open an issue with:

- **Use case** — what problem does this solve?
- **Current workaround** (if any)
- **Proposed solution** (optional)

Feature requests that align with the [architecture philosophy](#architecture)
have priority.

---

**This is not an AI. It is a bridge. And since v2.4, a platform.**

Thanks for reading. Now go break something and fix it.
