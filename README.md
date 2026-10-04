# TACLEX ML - Server Package

**Version:** v2.4.0-alpha
**License:** MIT
**Engine:** Arma 3 (Real Virtuality 4) — DayZ and Arma Reforger adapters ready

A multi-game mission generation platform that integrates Large Language Models
(LLMs) with Arma 3 through a lightweight HTTP bridge between SQF and a local
backend.

Not an AI running inside Arma 3. A bridge. And since v2.4, a platform.

## Overview

Every command typed by a player is intercepted by the client addon, sent
through a DLL bridge to a local HTTP backend, interpreted by an LLM using
native Tool Calling, converted to a structured PIPE command, and executed
in the game.

Heavy processing (token generation, JSON validation, name resolution) happens
entirely on the backend before reaching the engine. The game receives only a
short structured string, similar to a standard admin command.

Architecture (v2.4):

Chat -> SQF -> DLL -> HTTP -> TACLEX_CORE -> LLM -> ADAPTER -> PIPE -> SQF

Since v2.4, the CORE is 100% game-agnostic. Arma 3 is just one of its adapters.

## What This Is

- A multi-game mission generation platform (Arma 3 in production, DayZ and
  Reforger adapters architecturally proven)
- A programmer's experiment — not a commercial product
- An open-source learning project shared with the Bohemia community
- An alpha release: expect bugs and incomplete features

## What This Is Not

- An AI running inside Arma 3
- A commercial product
- A replacement for hand-crafted missions
- A client-only mod

## Architecture (v2.4)

Since v2.4, the project is split into three independent layers:

    TACLEX_CORE/       game-agnostic engine (11 modules)
    ADAPTERS/          one per game (arma3, dayz, reforger)
    CONTENT_PACKS/     JSON data per game (vanilla, RHS, CUP, DayZ)
    PROMPTS/           LLM prompt per game

The CORE contains zero Arma 3 knowledge. Adding a new game requires only a
new adapter — no modification to the CORE.

Proof of concept:
- load_adapter("arma3")     -> custom_mission     (PIPE)
- load_adapter("dayz")      -> dayz_mission       (PIPE)
- load_adapter("reforger")  -> reforger_mission   (JSON)

Same mission dict universal. Same execute(). Zero CORE changes.

Note: DayZ and Reforger adapters are architectural proofs. Their in-game
runtime (Enforce Script) is not yet implemented.

## Requirements

Server Host:
- Arma 3 Server (latest)
- CBA_A3 (latest)
- Windows 10 / 11 x64
- API key from Groq, OpenAI, Gemini, OpenRouter, or DeepSeek

Players:
- Arma 3 (latest)
- CBA_A3 (latest)
- TACLEX ML mod from Steam Workshop

## Installation

1. Download the latest release from the Releases page.

2. Copy @TACLEX_SERVER to:
   C:\Program Files (x86)\Steam\steamapps\common\Arma 3 Server\

3. Get an API key at groq.com (free, recommended).

4. Run TACLEX_Control.exe as Administrator.

5. Paste your API key, click Save .env, click Start Backend.

6. Edit taclax_admins.txt and add your SteamID64 (one per line).

7. Add @TACLEX_SERVER to server startup:
   -mod="@CBA_A3;@TACLEX_SERVER"

## Commands

Vehicles:
  spawn a kuma
  spawn a tempest
  spawn an apache
  spawn a littlebird
  spawn a hunter

Weather:
  set weather storm
  clear the weather
  make it rain
  set time to 22:30

Groups:
  spawn 4 enemies in front of me
  spawn 10 enemies near my position

Missions — Kill:
  kill the officer at 035049
  kill the officer at 035049 with ambush
  kill the officer at 035049, when he dies spawn escape vehicle
  kill the officer at 035049 with ambush, when half enemies die send wave

Missions — Rescue:
  rescue John at 035049
  resgata a Maria no porto

Missions — Destroy:
  destroy the truck at 035049
  destroy the antenna at 035049

Missions — Defend:
  defend the base at 035049 for 2 minutes
  defend the base at 035049 for 3 minutes

Missions — Multi-objective:
  kill the officer at 035049, extract at 040050
  escort the officer to 040050

Missions — Campaign (2 missions chained):
  kill the officer at 035049, then defend the base at 040050 for 2 minutes

Languages: English and Portuguese (auto-detected by the LLM).

## What's New in v2.4

### Platform Rebuild

- Backend split into 11 game-agnostic CORE modules
- ADAPTERS layer (arma3, dayz, reforger)
- Content packs per game
- Event bus (pub/sub)
- Rules engine (content-driven triggers)
- Campaign engine (persistent state)

### Mission System

- Briefing cinemático (on-screen, 12s)
- Visual log (MISSION COMPLETE / FAILED)
- Role-based units (1 officer + N guards)
- Hostage system
- Defend auto-wave (30s + 60s reinforcements)
- objective_complete trigger
- Runner paralelo (multi-task without blocking)
- 5 triggers: player_near, time_elapsed, unit_dead, percentage_dead,
  objective_complete
- 12 mission types

### Campaign Engine

- Persistent campaign state (campaign_state.json)
- Progressão M1 -> M2 -> M3
- split N missões
- Event-driven mission completion
- State variables between missions

### Content System

- Full RHS/CUP support (232 classnames)
- Context-aware classes (player side -> enemy classes)
- RHS MSV (player WEST -> enemy EAST)
- RHS USAF (player EAST -> enemy WEST)
- Vanilla fallback

### Dataset

- Event sourcing in Documents\Arma 3\TACLEX_dataset\
- missions.jsonl + results.jsonl
- Result queue (no more overwrite)

### Dev Tools

- build.ps1 (single command, 67s)
- core_test.py (8/8 decoupling tests)
- campaign_test.py (10/10 campaign tests)

## In-Game Example — Campaign

Player types in the TACLEX console (Insert key):

  kill the officer at 035049, then defend the base at 040050 for 1 minute

In-game result:

  Mission 1: "Officer Elimination"
    - Task + red marker at 035049
    - 5 enemy soldiers (RHS or vanilla, depending on pack)
    - Task completes when officer dies

  8 seconds pause

  Mission 2: "Base Defense" (auto-spawned)
    - Task + red marker at 040050
    - Wave 1 (30s): 4 reinforcements
    - Wave 2 (60s): 4 reinforcements
    - Task completes after timer

  Both missions share the same content pack classes.

## Supported Providers

- Groq (recommended, free)
- OpenAI (paid)
- Gemini (free tier limited)
- OpenRouter (free tier limited)
- DeepSeek (paid, cheap)

All support native Tool Calling.

## Configuration Files

All config files are auto-generated on first run of TACLEX_Control.exe.

taclax.env:
  LLM_PROVIDER=groq
  LLM_MODEL=openai/gpt-oss-120b
  LLM_ACTIVE_KEY=1
  LLM_API_KEY_1=your_key_here
  TACLEX_ADMIN_ONLY=false
  TACLEX_CONTENT_PACK=arma3_vanilla
  TACLEX_PROMPT_PACK=arma3_vanilla

taclax_admins.txt:
  one SteamID64 per line

taclax_admin_only.txt:
  true or false

## Troubleshooting

- Backend won't start: run as Administrator, check port 8000
- Commands return unknown: check API key
- Vehicle doesn't spawn: if RHS/CUP, ensure mod is loaded
- Classes are vanilla instead of RHS: check TACLEX_CONTENT_PACK in .env
- context deadline exceeded: provider too slow, switch to a faster model

## Known Limitations

- Alpha release, expect bugs
- DayZ and Reforger adapters are architectural proofs only
  (no runtime implemented yet)
- Arma 2 planned but not started
- Campaign state is global (not per-player yet)
- LLM provider: Groq only for now (fallback in roadmap)
- Onboarding requires manual .env setup
- Free LLM tiers have daily request limits

## Roadmap

- [x] Tool Calling integration
- [x] 232 vehicle enum
- [x] 12 mission types
- [x] Runner paralelo (multi-task)
- [x] Campaign engine (2+ missions)
- [x] Briefing + log visual
- [x] CORE agnostic + ADAPTERS
- [x] Multi-game proof (arma3 + dayz + reforger)
- [ ] Full 12 objectives in-game test
- [ ] Steam Workshop release
- [ ] API key onboarding wizard
- [ ] Fallback provider (Groq -> DeepSeek)
- [ ] DayZ runtime (Enforce Script)
- [ ] Reforger runtime (Enfusion)

## License

MIT License. Free to use, modify, distribute and sublicense.
Attribution required.

## Disclaimer

TACLEX ML is an educational and experimental project. It is not sold,
licensed, or monetized.

Use at your own risk. The author is not responsible for any in-game behavior
generated by LLM output.

This is not an AI. It is a bridge. And since v2.4, a platform.
