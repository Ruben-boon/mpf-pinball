# Architecture

How configuration loads, how the modes fit together, and the central
**mansion / center-scoop** progression that drives this game.

## Config load order

`config/config.yaml` is the entry point. It `config:`-includes the machine-wide files
(in this order) and registers the `modes:` that run:

```
config/config.yaml
├── hardware.yaml    OPP platform, ports, PSU
├── switches.yaml    every switch (s_*)
├── coils.yaml       every coil (c_*)
├── lights.yaml      every LED (led_*)
├── devices.yaml     flippers, autofire coils, ball devices
├── images.yaml      DMD images
├── displays.yaml    window + dmd displays
├── slides.yaml      slide definitions + slide_player
└── sounds.yaml      sound assets (music/voice/sfx)
```

`config.yaml` also defines the **sound_system** tracks and the `playfield` playfield
(default source device `bd_trough`).

Registered modes (order in `config.yaml`):
`base, attract, grave_targets, left_orbit, mansion, center_scoop_lit`.

> `modes/test_mode/` exists on disk but is **empty and not registered** — it does nothing.

## Mode priorities (higher overrides lower)

| Mode | Priority | Starts on | Stops on |
|---|---|---|---|
| `center_scoop_lit` | 1002 | `ball_starting` | `ball_will_end` |
| `mansion` | 1000 | `ball_starting` | `ball_will_end` |
| `base` | 250 | `ball_starting` | (runs through ball) |
| `grave_targets` | 200 | `start_mode_grave` | `grave_targets_complete`, `ball_will_end` |
| `left_orbit` | 200 | `start_mode_orbit` | `left_orbit_complete`, `ball_will_end` |
| `attract` | (n/a) | when idle | when a game starts |

## The mansion / center-scoop game loop (the core)

The center scoop is the **starter** of the game. Shooting the lit ("plugged in")
scoop starts the next mansion mode. Each mansion mode maps to a mansion LED
("window"). Two modes exist today: `grave_targets` (window `led_mansion_1`) and
`left_orbit` (window `led_mansion_2`).

### Two states per window: MARKED vs DONE

- **MARKED** (window solid) = the mode was **started**. Marked the instant the mode
  starts. Once marked, the window never comes up as "next" again — even if the player
  drains mid-mode without finishing.
- **DONE** (objective completed) = the **reward** was earned (bonus score + sound).
  A window can be MARKED but not DONE (player drained before completing).

Nothing carries across balls — all state resets on `ball_starting`.

### The orchestrator: `mansion` mode (priority 1000)

`mansion` (was `ship_modes`) owns the rotation state via player variables:
`grave_marked`, `orbit_marked`, `grave_done`, `orbit_done`, and the string pointer
`next_mode` (`grave` / `orbit` / `idle`).

- **Pointer** (`recompute_next_mode`): resolves to the first *unmarked* window
  (grave → orbit → `idle` when all marked). The pointed window **blinks**
  (`mansion_blink`); marked windows are driven **solid** (`led_mansion_solid`),
  both keyed `mansion_N_show` so the solid replaces the blink.
- **Slingshot rotation**: `s_*_slingshot_active → rotate_next_mode` toggles the
  pointer to the *other* unmarked window. With one unmarked window left the target is
  marked, the guard fails, and the pointer stays put.

> ⚠️ The "nothing left" sentinel is the string **`idle`**, NOT `none`. MPF's YAML
> loader coerces `none`/`null` (even quoted) to null, and `variable_player` then
> rejects it (`CFE-variable_player-2`). Confirmed crash + fix.

### The scoop: `center_scoop_lit` mode (priority 1002)

Runs the whole ball so it can always arbitrate scoop hits. Tracks `scoop_plugged`
(0/1).

- **Plugged + a mode pending** → shooting `s_center_scoop` posts `start_mode_grave`
  / `start_mode_orbit`, which (a) marks the window in `mansion`, (b) starts the
  objective mode, (c) **unplugs** the scoop (`led_center_scoop` off).
- **Unplugged** → scoop hit plays the `taf_not_plugged_in` ("it's not plugged in
  yet") callout and does nothing else.
- **Re-plug**: hitting the bear / main ramp (`s_right_ramp_entry`) while unplugged
  posts `scoop_replugged` → re-lights the scoop so it can start the next window.
- **All windows marked** (`next_mode == idle`) → scoop stays dark/inert (end state
  for now; wizard mode later).

```
ball_starting
   │  scoop plugged (red), mansion pointer blinks led_mansion_1 (grave = next)
   ▼
s_center_scoop (plugged) ──► start_mode_grave
        ├─ mansion: grave_marked=1, led_mansion_1 solid, pointer → orbit (blinks _2)
        ├─ grave_targets mode starts (spell G-R-A-V-E)
        └─ center_scoop_lit: scoop unplugged (led_center_scoop off)
   │
   │  (objective runs; completing it posts grave_targets_complete → reward + DONE)
   ▼
s_right_ramp_entry (bear ramp) ──► scoop_replugged ──► scoop re-lit
   │
   ▼
s_center_scoop (plugged) ──► start_mode_orbit ──► … (left_orbit, window _2) …
```

### Scoop audio + timing

- `s_center_scoop_active → taf_mansion_1` (mansion capture callout) fires
  immediately, owned by `base`.
- The ball is held in the scoop **3s** before ejecting
  (`bd_center_scoop` `eject_events: s_center_scoop_active:3s`).
- Each objective mode plays its own start callout **2s** after start
  (`mode_<name>_started` → delayed `*_mode_start_callout` event → sound): grave =
  `taf_start_grave` (seance), orbit = `taf_start_orbit` (straight to the vault).

## Sound system

Three tracks (defined in `config/config.yaml`):

| Track | Simultaneous | Volume | Purpose |
|---|---|---|---|
| `music` | 1 | 0.5 | background music, loops |
| `voice` | 1 | 0.7 | callouts / announcements |
| `sfx` | 8 | 0.4 | slings, pops, hits |

Assets default to `load: on_demand`. Files live under `sounds/{music,voice,sfx}/`.

## Event-driven model

Everything is events. Switches post `<switch>_active`; shots post `<shot>_hit` (and
profile-state variants like `<shot>_lit_hit`); counters post completion events; modes
post lifecycle events (`mode_<name>_started/_ended/_will_start`). Config "players"
(`event_player`, `show_player`, `sound_player`, `light_player`, `variable_player`,
`slide_player`) react to events. See `docs/mpf-reference.md` for the official pages.
