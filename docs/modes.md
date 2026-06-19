# Modes

Per-mode reference, grounded in the actual `modes/<name>/config/<name>.yaml` files.
For the cross-mode progression see `docs/architecture.md` (ship-modes flow).

## base — priority 250, starts `ball_starting`

The always-on gameplay mode. Defines the shared shot vocabulary the other modes reuse.

- **counters:** `right_ramp_counter`, `top_loop_counter` (both persist, start enabled).
- **sequence_shots** (shared by other modes): `shot_right_ramp`
  (`s_right_ramp_entry → s_ramp_top`), `shot_left_ramp`, `shot_top_loop`,
  `shot_left_orbit` (`s_upper_left_loop → s_upper_right_loop`), `shot_right_orbit`
  (reverse). All 3s timeout.
- **variable_player:** pops +200, slings +50, it-hole +200, ramp/scoop hits +100.
- **show_player:** pop hits blink `led_gy`.
- **light_player:** `mode_base_started` turns the GI blue/white; `mode_base_ended` off.
- **sound_player:** stops startup music & plays `music_prelaunch` on start; on first
  `playfield_active` switches to `dragon_roost`; sling/pop sfx.
- **display:** builds the DMD game slide (score, `P(number)`, `B(ball)`).

> Lots of inlane/outlane shots are present but **commented out**. Don't treat them as live.

## attract — runs when idle

Pure light-show + music loop; no scoring.
- **show_player:** a self-chaining sequence — each show fires the next via
  `events_when_completed` (`bottom_top → blue_whirlpool → red_rotate_center →
  riple_target_a → yellow_top_bottom → center_blue_swirl → right_left → riple_target_r →
  riple_target_e → riple_target_g → back to bottom_top`).
- **sound_player:** `song_of_storms` looping (`loops: -1`) on `mode_attract_started`.
- Mode-local shows live in `modes/attract/shows/`.

## pop_bumpers — priority 200, objective mode

- **counter `pop_bumper_counter`:** counts `s_pop1..5_active`, completes at **20**,
  posts `pop_bumpers_20_hits_reached, pop_mode_complete`. Resets on `ball_starting`.
- **stop_event:** `ship_pop_mode_complete` (fires after collection — see architecture).
- +2000 on completion. Achievement LED: `led_mansion_1`.

## grave_targets — priority 200, objective mode

Sequential G-R-A-V-E lit-shot chain using a shared `target_profile`
(states off→lit→down, `advance_on_hit: false`, advanced via events).

- **order:** `shot_target_g → shot_target_r → shot_target_a → shot_right_ramp
  (s_ramp_top) → shot_right_scoop`. Each shot's `advance_events` chains off the previous
  shot's `*_lit_hit`.
- `shot_right_scoop_lit_hit` posts `grave_targets_complete` (+2000). LED: `led_mansion_2`.
- Per-target ripple shows + `sword_slice` sfx on hits.
- **stop_event:** `ship_grave_mode_complete`.

## ramp_alternate — priority 200, objective mode

Alternating ramp↔jump sequence using `ramp_alternate_profile`.
- **shots:** `shot_ramp_alternate_ramp` (`s_ramp_top`, led_ramp_2) and
  `shot_ramp_alternate_jump` (`s_eject_lane`, led_jump_multiball).
- `mode_..._started → reset_ramp_alternate_sequence → advance_ramp_alternate_ramp`.
- Hitting the lit jump posts `ramp_alternate_sequence_complete` + resets; the mode-complete
  path posts `ramp_alternate_complete` (+5000). LED: `led_mansion_3`.

## ghost_ship — priority 200, objective mode

- **counter `ghost_ship_counter`:** counts `shot_left_orbit_hit`, completes at **2**,
  posts `ghost_ship_mode_complete → ghost_ship_complete` (+3000). LED: `led_mansion_4`.
- Left-orbit hits +1000, blink `led_advance`; solid on complete.
- **stop_event:** `ship_ghost_ship_mode_complete`.

## left_ramp_mode — priority 200, objective mode

- **counter `left_ramp_counter`:** counts `shot_left_ramp_hit`, completes at **2**,
  posts `left_ramp_mode_complete → left_ramp_complete` (+3000). LED: `led_mansion_5`.
- Ramp hits +1000, blink `led_upper_ramp_star`; solid on complete.
- **stop_event:** `left_ramp_mode_complete`.

## ship_modes — priority 1000, starts `ball_starting`, stops `ball_will_end`

The meta-mode. Listens for the five `*_complete` events, re-posts `*_ready` to start the
matching **achievement** (mansion LED 1–5). Achievements collect at the center scoop and,
when completed, post `ship_*_mode_complete` (the objective modes' stop events). Full flow:
`docs/architecture.md`. Mode-local shows in `modes/ship_modes/shows/`.

## center_scoop_lit — priority 1002

The shared collector. Starts on `light_center_scoop`, stops on `unlight_center_scoop`.
- Lights `led_center_scoop` red on start; off when `shot_center_scoop_hit`.
- `shot_center_scoop` watches `s_center_scoop`; on hit posts **all** `ship_*_collected`
  events + `ship_mode_complete_bonus` (+20000) + `unlight_center_scoop`.

## test_mode — empty

`modes/test_mode/` has no config and is not registered in `config.yaml`. Inert.
