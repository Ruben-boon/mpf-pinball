# Architecture

How configuration loads, how the modes fit together, and the central
**ship-modes / achievement** progression that drives this game.

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
`base, attract, grave_targets, pop_bumpers, ramp_alternate, ghost_ship,
left_ramp_mode, ship_modes, center_scoop_lit`.

> `modes/test_mode/` exists on disk but is **empty and not registered** — it does nothing.

## Mode priorities (higher overrides lower)

| Mode | Priority | Starts on | Stops on |
|---|---|---|---|
| `center_scoop_lit` | 1002 | `light_center_scoop` | `unlight_center_scoop` |
| `ship_modes` | 1000 | `ball_starting` | `ball_will_end` |
| `base` | 250 | `ball_starting` | (runs through ball) |
| `grave_targets` | 200 | `ball_starting` | `ship_grave_mode_complete` |
| `pop_bumpers` | 200 | `ball_starting` | `ship_pop_mode_complete` |
| `ramp_alternate` | 200 | `ball_starting` | (sequence completion) |
| `ghost_ship` | 200 | `ball_starting` | `ship_ghost_ship_mode_complete` |
| `left_ramp_mode` | 200 | `ball_starting` | `left_ramp_mode_complete` |
| `attract` | (n/a) | when idle | when a game starts |

## The ship-modes progression (the core game loop)

This is the most important and least obvious part of the project. There are **three
stages**: complete a mode → light & collect it at the center scoop → achievement done.
Each of the five "objective" modes maps to a mansion LED.

### Stage 1 — complete an objective mode

Each objective mode does its own thing, then posts a **`*_complete` event**:

| Objective mode | Goal | Posts |
|---|---|---|
| `pop_bumpers` | 20 pop hits (`pop_bumper_counter`) | `pop_mode_complete` |
| `grave_targets` | G-R-A-V-E target/ramp/scoop sequence | `grave_targets_complete` |
| `ramp_alternate` | ramp↔jump sequence | `ramp_alternate_complete` |
| `ghost_ship` | 2× left orbit (`ghost_ship_counter`) | `ghost_ship_complete` |
| `left_ramp_mode` | 2× left ramp (`left_ramp_counter`) | `left_ramp_complete` |

### Stage 2 — ship_modes arms an achievement

`ship_modes` (priority 1000) listens for those `*_complete` events and re-posts a
`*_ready` event, which **starts** the matching achievement:

```
pop_mode_complete       → ship_pop_mode_ready            → start achievement ship_mode_pop  (led_mansion_1)
grave_targets_complete  → ship_grave_mode_ready          → start achievement ship_mode_grave (led_mansion_2)
ramp_alternate_complete → ship_ramp_alternate_mode_ready → ship_mode_ramp_alternate (led_mansion_3)
ghost_ship_complete     → ship_ghost_ship_mode_ready     → ship_mode_ghost_ship (led_mansion_4)
left_ramp_complete      → ship_left_ramp_mode_ready      → ship_mode_left_ramp (led_mansion_5)
```

When an achievement **starts** it:
- runs `show_blink` on its mansion LED (`show_when_started`), and
- posts `light_center_scoop` (`events_when_started`).

### Stage 3 — collect at the center scoop

`light_center_scoop` starts the `center_scoop_lit` mode (priority 1002), which lights
`led_center_scoop` red. Hitting `s_center_scoop` fires `shot_center_scoop_hit`, and
`center_scoop_lit` blindly posts **all** collect events:

```
shot_center_scoop_hit →
  ship_pop_mode_collected
  ship_grave_mode_collected
  ship_ramp_alternate_mode_collected
  ship_ghost_ship_mode_collected
  ship_left_ramp_mode_collected
  ship_mode_complete_bonus      (+20000)
  unlight_center_scoop          (stops center_scoop_lit)
```

Each `*_collected` is the `complete_events` of its achievement. Only achievements that
are currently *started* react — the rest ignore the collect event. The completing
achievement turns its mansion LED solid (`show_when_completed: on`) and posts its
`ship_*_mode_complete` event, which is the **stop_event** for the original objective mode.

```
ball_starting
   │  (objective mode runs, e.g. pop_bumpers)
   ▼
pop_mode_complete ──► ship_pop_mode_ready ──► achievement ship_mode_pop starts
                                                 │ blink led_mansion_1
                                                 └─ light_center_scoop ──► center_scoop_lit
                                                                              led_center_scoop = red
s_center_scoop ──► shot_center_scoop_hit ──► ship_pop_mode_collected ──► achievement completes
                                                 │ led_mansion_1 solid
                                                 └─ ship_pop_mode_complete ──► stops pop_bumpers mode
```

> Note: `center_scoop_lit` posts every collect event on every scoop hit. That works
> because non-started achievements ignore their collect event, but it means the scoop is
> a shared collector — keep that in mind before adding scoop-driven logic elsewhere.

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
