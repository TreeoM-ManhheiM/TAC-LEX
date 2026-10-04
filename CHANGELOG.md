# Changelog

All notable changes to TACLEX ML are documented here.

## [v2.4.0-alpha] - 2026-10-04

Platform Rebuild: CORE + Multi-Game + Campaign Engine.

This is the largest update since launch. TACLEX stopped being
"just another dynamic mission mod for Arma 3" and became a
multi-game mission generation platform.

### Architecture — the biggest change

REBUILT:
- Monolithic backend (962 lines) split into 11 CORE modules
- CORE is now 100% game-agnostic (zero Arma 3 references)
- Every game-specific concept moved to ADAPTERS

CORE modules:
  parser, validator, content_loader, prompt_loader,
  dataset_logger, event_bus, rules_engine, executor,
  llm_adapter, http_server, campaign_engine

ADAPTERS:
- arma3     -> custom_mission     (PIPE format)
- dayz      -> dayz_mission       (PIPE format)   [adapter ready]
- reforger  -> reforger_mission   (JSON format)   [adapter ready]

Proof: the same mission dict universal works on all three.
Zero modifications to CORE to add a new game.

### Mission System

ADDED:
- Briefing cinemático (on-screen, 12s)
- Visual log (MISSION COMPLETE / FAILED on screen)
- Role-based units (1 officer + N guards)
- Hostage system (rescue, follow, extract)
- Defend auto-wave (30s + 60s reinforcements)
- objective_complete trigger (wait for specific task)
- Runner paralelo (multi-task without blocking)
- 5 triggers: player_near, time_elapsed, unit_dead,
  percentage_dead, objective_complete
- 12 mission types fully validated

FIXED:
- Trigger scope leaks (_sideStr, _enemySide, _clsSoldier)
- M2 class propagation in campaigns (was fallback to vanilla)
- Defender auto-wave class from content pack
- Spawn classes side-aware via PIPE

### Campaign Engine (new module)

ADDED:
- Persistent campaign state (campaign_state.json)
- Progressão M1 -> M2 -> M3
- split N missões (era hardcoded 2)
- Event-driven mission completion
- State variables (officer_alive, radar_destroyed, etc)

TESTED: 10/10 in isolated test suite.

### Content System

REBUILT:
- Content Packs: arma3_vanilla, rhs, cup, dayz
- Prompt Packs: arma3_vanilla, dayz
- Full RHS/CUP support (232 classnames)
- Context-aware classes (player side -> enemy classes)

NEW:
- RHS MSV (player WEST -> enemy EAST)
- RHS USAF (player EAST -> enemy WEST)
- Vanilla fallback for missing mods

### Dataset & Telemetry

ADDED:
- Event sourcing (missions.jsonl + results.jsonl)
- Stored in Documents\Arma 3\TACLEX_dataset\
- mission.result queue (não sobrescreve mais)
- 4 facções testadas (BLUFOR, OPFOR, IND, CIV)

### Build & Dev Tools

ADDED:
- build.ps1 (1 comando, 67s, build + copy + report)
- core_test.py (8/8 decoupling tests)
- campaign_test.py (10/10 campaign tests)

### Known Limitations

- DayZ and Reforger adapters are architectural proofs only.
  Runtime (Enforce Script) not yet implemented.
- Arma 2 planned but not started.
- Campaign state is global (not per-player yet).
- LLM provider: Groq only (fallback provider in roadmap).
- Onboarding requires manual .env setup.

### Roadmap (next 30 days)

1. Full 12 objectives in-game test (BLUFOR/OPFOR/IND)
2. Steam Workshop release
3. API key onboarding wizard
4. Fallback provider (Groq -> DeepSeek)

---

## [v1.1.1-alpha] - 2026-09-28

Grid fix + ACE/VCOM + RHS/CUP.

FIXED:
- Grid now uses 100m cell (works on ANY map)
- spawn_group with caller validation
- Client PBO cleaned of duplicate functions
- System message loop filtered

ADDED:
- 12 mission types
- Full RHS/CUP support (232 classnames)
- Vanilla fallback for missing mods
- ACE3 + VCOM AI integration

## [v1.1.0-alpha] - 2026-09-28

Grid coordinates + RHS/CUP + 12 mission types.

NEW:
- Grid coordinates for any map (at 045-052, at 100200)
- RHS/CUP vehicle map rebuilt with 232 entries
- 12 mission types (added capture, secure, escort, investigate,
  survive, deliver)
- Backend extracts grid from natural text (regex fallback)
- Renderer passes position in PIPE

TESTED: 8/8 commands via HTTP with grid working.

## [v1.0.0-alpha] - 2026-09-27

First public release of TACLEX ML.

- Native Tool Calling support via local HTTP backend
- 232 vehicle enum (vanilla + RHS + CUP classnames)
- Custom mission generation (kill, rescue, destroy, defend)
- Weather and time commands
- Enemy group spawning
- Multi-language input (English and Portuguese)
- Support for Groq, OpenAI, Gemini, OpenRouter, DeepSeek

Architecture:
Chat -> SQF -> DLL -> HTTP -> LLM -> JSON -> Renderer -> PIPE -> SQF
