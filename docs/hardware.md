# Hardware

Physical machine wiring as defined in `config/hardware.yaml`, `switches.yaml`,
`coils.yaml`, `lights.yaml`, `devices.yaml`. This documents what's in the config — verify
against the physical machine before changing addresses.

## Platform

```yaml
hardware: { platform: opp, driverboards: gen2 }
opp:      { ports: com3, com4 }
psus:     { default: { release_wait_ms: 50 } }
```

Two OPP Gen2 chains on `com3` and `com4`.

## Address format

`board-wing-channel`, e.g. `0-0-19`, `1-0-2`.
- **Board 0** = first chain, **Board 1** = second chain.
- Switch and coil/LED channel spaces are independent (a switch `1-0-2` and an LED
  `1-0-2` are different physical things on the same board).

## Switches (`s_*`)

| Switch | Number | Tags / note |
|---|---|---|
| s_left_flipper | 0-0-19 | left_flipper |
| s_right_flipper | 0-0-17 | right_flipper |
| s_left_slingshot | 0-0-26 | left_slingshot |
| s_right_slingshot | 0-0-24 | right_slingshot |
| s_left_outlane | 1-0-9 | |
| s_target_g | 1-0-2 | playfield_active |
| s_target_r | 1-0-8 | playfield_active |
| s_target_a | 1-0-44 | playfield_active |
| s_right_scoop | 1-0-10 | |
| s_center_scoop | 1-0-11 | collect point (ship modes) |
| s_it_hole | 0-0-10 | |
| s_pop1..s_pop5 | 0-0-2, 0-0-1, 0-0-9, 0-0-3, 0-0-8 | pop bumpers |
| s_trough1 | 1-0-1 | only one trough switch wired |
| s_startButton | 1-0-3 | start |
| s_right_ramp_entry | 1-0-41 | |
| s_left_ramp_entry | 1-0-42 | |
| s_ramp_top | 1-0-43 | shared ramp-top switch |
| s_top_loop_entry | 1-0-60 | |
| s_upper_right_loop | 1-0-39 | |
| s_upper_left_loop | 1-0-38 | |
| s_eject_lane | 1-0-37 | |
| s_eject_hole | 1-0-64 | |

**Disabled (commented out) in `switches.yaml`:** inlanes/outlanes
`s_right_inlane`, `s_right_outlane`, `s_left_inlane_1`, `s_left_inlane_2`; extra trough
switches `s_trough2..5`; and a large block of `s_test_*` (1-0-32…95). Don't reference
these as if active.

## Coils (`c_*`)

| Coil | Number | Pulse / note |
|---|---|---|
| c_flipper_left_main | 0-0-4 | 20ms |
| c_flipper_left_hold | 0-0-5 | allow_enable |
| c_flipper_right_main | 0-0-7 | 20ms |
| c_flipper_right_hold | 0-0-15 | allow_enable |
| c_left_slingshot | 0-0-2 | 12ms @ 0.70 power |
| c_right_slingshot | 0-0-3 | 12ms @ 0.70 power |
| c_flipper_top_main | 0-0-1 | 25ms — *flagged "currently disabled" in config comment* |
| c_flipper_top_hold | 0-0-6 | hold 0.20 |
| c_flipper_little_main | 1-0-3 | 10ms |
| c_flipper_little_hold | 1-0-1 | hold 0.20 |
| c_pop1..c_pop5 | 0-0-8, 0-0-0, 0-0-11, 0-0-14, 0-0-12 | 17ms each |
| c_center_scoop | 1-0-5 | 20ms |
| c_right_scoop | 1-0-2 | 80ms |
| c_it_hole | 1-0-7 | 80ms |
| c_trough_eject | 1-0-4 | 35ms |

> Note `c_flipper_top_*` / `c_flipper_little_*` reuse board-1 channels (1-0-3, 1-0-1)
> that also appear as switch numbers — that's expected (separate switch vs driver spaces).
> The top flipper coils are marked as problem/disabled in the config comments even though
> the `top_flipper` device is still defined in `devices.yaml`.

## Lights (`led_*`)

All LEDs are `subtype: led, type: rgb`. They're grouped in `lights.yaml` by physical
chain:
- **GI lower half:** `led_gi_*` (ball guides, slingshots, pops, gr-targets, smallflip)
- **Lower lanes:** `led_left_outlane/inlane_1/inlane_2`, `led_right_outlane/inlane`
- **Mansion:** `led_mansion_1 … led_mansion_14` — `1–5` are the ship-modes achievement LEDs
- **Middle:** `led_letter_g/r/d/e/a/v`, `led_thunder_*`, `led_the_power`, `led_swamp_1..5`
- **Bottom chain:** `led_greed_1..5`, `led_ramp_1/2`, `led_it`, `led_loop`,
  `led_upper_ramp_*`, `led_jump_*`, `led_center_scoop`, `led_advance`, `led_gy`, etc.

(Full channel list is in `config/lights.yaml`; reproduce only when you need a specific
channel.)

## Devices (`config/devices.yaml`)

- **Flippers:** `left_flipper`, `right_flipper`, `top_flipper`, `little_flipper` — each
  main+hold coil, activation switch, `enable_events: machine_reset_phase_3`.
  (`top_flipper` and `little_flipper` share the main flipper switches.)
- **Autofire coils:** `left_slingshot`, `right_slingshot`, `pop1..pop5` — fire their coil
  on their switch automatically; enable on `machine_reset_phase_3`.
- **Ball devices:**
  - `bd_trough` — switch `s_trough1`, eject `c_trough_eject`, tags `trough, home, drain`
  - `bd_center_scoop` — `s_center_scoop` / `c_center_scoop`, eject_timeout 3s
  - `bd_right_scoop` — `s_right_scoop` / `c_right_scoop`, 3s
  - `bd_it_hole` — `s_it_hole` / `c_it_hole`, 1s
