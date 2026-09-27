# TACLEX ML - Server Package

**Version:** v1.0.0-alpha
**License:** MIT
**Engine:** Arma 3 (Real Virtuality 4)

A technical experiment that integrates Large Language Models (LLMs) into Arma 3 through a lightweight HTTP bridge between SQF and a local backend.

This is a programmer's experiment — not a commercial product. It is not an AI running inside Arma 3. It is a bridge.

## Overview

Every command typed by a player is intercepted by the client addon, sent through a DLL bridge to a local HTTP backend, interpreted by an LLM using native Tool Calling, converted to a structured PIPE command, and executed in the game.

Heavy processing (token generation, JSON validation, name resolution) happens entirely on the backend before reaching the engine. The game receives only a short structured string, similar to a standard admin command.

Architecture:

Chat -> SQF -> DLL -> HTTP -> LLM (Tool Calling) -> Renderer -> PIPE -> SQF

## What This Is

- A programmer's experiment — not a commercial product
- A demonstration of LLM integration with the Real Virtuality engine
- An open-source learning project shared with the Arma 3 community
- An alpha release: expect bugs and incomplete features

## What This Is Not

- An AI running inside Arma 3
- A commercial product
- A replacement for hand-crafted missions
- A client-only mod

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

Missions — Rescue:
  rescue John at the port
  rescue John at the port with 30 enemies
  resgata a Maria no porto com 30 inimigos
  rescue the hostage in Zaros

Missions — Kill:
  kill the officer in Pyrgos
  kill the enemy commander in Kavala

Missions — Destroy:
  destroy the antenna
  destroy the radio tower in Kavala

Missions — Defend:
  defend the base for 5 minutes

Languages: English and Portuguese (auto-detected by the LLM).

## In-Game Examples

Example 1 — Rescue with 30 enemies

Player types in the TACLEX console (Insert key):

  rescue John at the port with 30 enemies

Backend log:

  >> [V2] tool=custom_mission args={
       'mission_name': 'Rescue John at Port',
       'briefing': 'The team must infiltrate the harbor, locate
                    the hostage John, and extract him safely.',
       'objectives': [
         {'type': 'move',    'value': 'reach the port'},
         {'type': 'rescue',  'value': 'John'},
         {'type': 'extract', 'value': 'John to safe zone'}
       ]
     }
  >> resposta: OK|custom_mission|Rescue John at Port|...|move:reach the port;rescue:John;extract:John to safe zone

In-game result:
  - 1 task with 3 subtasks appears on the map
  - 30 enemy soldiers spawn at a safe position ~400m away
  - 1 hostage (civilian) spawns at the target location
  - A red marker marks the objective area
  - Mission only completes when all objectives are met

Example 2 — Assassinate an officer

Player types:

  kill the officer in Pyrgos

In-game result:
  - Task appears: "Eliminate Officer in Pyrgos"
  - 5 enemy soldiers spawn at ~400m
  - Red marker on the map
  - Task completes when all 5 enemies are eliminated

Example 3 — Destroy a target

Player types:

  destroy the antenna

In-game result:
  - Task appears: "Antenna Demolition"
  - 5 enemy soldiers spawn at the target area
  - Red marker + flag object at the location
  - Task completes when all enemies are eliminated

Example 4 — Defend for 5 minutes

Player types:

  defend the base for 5 minutes

In-game result:
  - Task appears: "Base Defense"
  - Timer counts down from 60 seconds
  - Task completes automatically after the timer expires

## Architecture

Chat -> SQF -> DLL -> HTTP -> LLM -> Renderer -> PIPE -> SQF -> Game

Components:
- SQF: chat interception, dispatch
- DLL: HTTP bridge (Go)
- Backend: LLM orchestration (Python)
- Renderer: PIPE formatting
- LLM: remote natural language interpretation

## Supported Providers

- Groq (recommended, free, 14,400 req/day)
- OpenAI (paid)
- Gemini (20 req/day free)
- OpenRouter (50 req/day free)
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

taclax_admins.txt:
  one SteamID64 per line

taclax_admin_only.txt:
  true or false

## Troubleshooting

- Backend won't start: run as Administrator, check port 8000
- Commands return unknown: check API key
- Vehicle doesn't spawn: if RHS/CUP, ensure mod is loaded
- Mission completes immediately: update to v1.0.0-alpha or newer
- context deadline exceeded: provider too slow, switch to a faster model

## Known Limitations

- Alpha release, expect bugs
- RHS and CUP vehicles require their mods to be loaded
- Mission quality depends on LLM output and may vary
- No client-only installation mode
- Free LLM tiers have daily request limits

## Roadmap

- [x] Tool Calling integration
- [x] 232 vehicle enum
- [x] Custom missions (kill, rescue, destroy, defend)
- [ ] Additional mission types
- [ ] isClass validation for missing mods
- [ ] In-game admin panel

## License

MIT License. Free to use, modify, distribute and sublicense. Attribution required.

## Disclaimer

TACLEX ML is an educational and experimental project. It is not sold, licensed, or monetized.

Use at your own risk. The author is not responsible for any in-game behavior generated by LLM output.

This is not an AI. It is a bridge.
