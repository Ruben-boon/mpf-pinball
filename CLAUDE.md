# CLAUDE.md

Guidance for Claude Code working in this repository. This file is the **router**:
it holds the facts that are true every session and points to deeper docs in `docs/`.
Read the relevant `docs/` file before doing real work on that area.

## What this is

A **Mission Pinball Framework (MPF)** project that runs a physical pinball machine.
MPF is a Python framework driven entirely by YAML config files — it handles hardware
I/O, game logic, modes, scoring, displays, sound, and LED shows.

- **MPF version:** 0.57.3 (the **0.57 LTS** line — *not* the 0.80 "latest" line)
- **config_version:** 6 (every config file starts with `#config_version=6`)
- **show_version:** 6 (every show file starts with `#show_version=6`)
- **Hardware:** OPP (Open Pinball Project) Gen2 driverboards on serial ports `com3, com4`
- **Display:** 128×32 DMD (`dmd`), shown in a 1280×320 window via a `color_dmd` effect

## Commands

```bash
mpf both -X -vV        # run game + media controller in SMART VIRTUAL mode (no hardware) — use this to test
mpf both -vV           # run on real hardware, verbose
mpf both               # normal operation
mpf game -b -vV        # game engine only, no media controller
tail -100 logs/*.log   # check latest logs after a change
```

**Always validate config changes with `mpf both -X -vV` and check the logs before
suggesting they run on hardware.** Virtual mode needs no boards attached.

## Where to find things

| You need… | Look in |
|---|---|
| How configs load + the ship-modes / achievement flow | `docs/architecture.md` |
| Boards, address format, full switch/coil/LED inventory | `docs/hardware.md` |
| What each mode does (per-mode reference) | `docs/modes.md` |
| Sound/music wiring, tracks, audio gotchas | `docs/audio.md` |
| Official MPF docs for a feature we use (versioned links) | `docs/mpf-reference.md` |
| Machine config entry point | `config/config.yaml` |
| A specific mode | `modes/<name>/config/<name>.yaml` |
| Authoring shows + the showcreator editor workflow | `docs/shows.md` |
| Light shows (global) | `shows/` |
| Light shows (mode-local) | `modes/<name>/shows/` |

## Answering MPF questions — source of truth

1. **This repo's actual configs** are the ground truth for *how this machine behaves*.
   Read the real YAML; don't assume. Many sections have commented-out / disabled blocks
   (see `docs/hardware.md` "Disabled" notes) — don't treat commented config as active.
2. **For MPF rules/syntax**, cite the **official 0.57 LTS docs**, not memory. The links in
   `docs/mpf-reference.md` point at `missionpinball.org/latest/…`; when a feature's
   behavior could differ between LTS (0.57) and latest (0.80), verify against 0.57.
3. If repo behavior and the docs disagree, say so — don't silently "fix" working config.

## Recording confirmed fixes

When a change has been tested **and** the user confirms it works ("it works", "it's fixed",
etc.), record the confirmed behavior in the matching `docs/` file (e.g. audio →
`docs/audio.md`, modes → `docs/modes.md`). Capture the working wiring and any non-obvious
gotcha that was learned. Only document things that are actually confirmed — unconfirmed
attempts don't go in the docs.

## Conventions

- **Naming:** switches `s_*`, coils `c_*`, lights `led_*`, shots `shot_*`, shows are
  descriptive (`blue_whirlpool`, `show_blink`).
- **Hardware addresses:** `board-wing-channel`, e.g. `0-0-19`, `1-0-2`.
- **YAML:** 2-space indent, no tabs. Every config file opens with `#config_version=6`.
- **Editing:** prefer editing existing config files over creating new ones; register any
  new mode in `config/config.yaml` under `modes:`.

## Safety notes (physical machine)

- Coils carry `default_pulse_ms` (≈10–80ms) and power limits — don't raise these casually;
  over-driving a coil can damage it. Hold coils use `allow_enable: true` / low `default_hold_power`.
- Ball devices need their switch + eject coil + sane `eject_timeouts`.
- Flippers, slingshots, and pop bumpers enable on `machine_reset_phase_3`.
