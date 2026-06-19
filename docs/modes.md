# Modes

Per-mode reference, grounded in the actual `modes/<name>/config/<name>.yaml` files.
For the cross-mode progression see `docs/architecture.md` (mansion / center-scoop flow).

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

## grave_targets — priority 200, mansion mode (window `led_mansion_1`)

Started by the **center scoop** (`start_events: start_mode_grave`), not `ball_starting`.
Sequential G-R-A-V-E lit-shot chain using a shared `target_profile`
(states off→lit→down, `advance_on_hit: false`, advanced via events).

- **order:** `shot_target_g → shot_target_r → shot_target_a → shot_right_ramp
  (s_ramp_top) → shot_right_scoop`. Each shot's `advance_events` chains off the previous
  shot's `*_lit_hit`.
- `shot_right_scoop_lit_hit` posts `grave_targets_complete` (+2000) = DONE/reward.
- Per-target ripple shows. **SFX:** target hits play the ha/ho laugh pool (`taf_pop`).
- **Start callout:** `taf_start_grave` (seance) 2s after start (see architecture).
- **stop_events:** `grave_targets_complete, ball_will_end`.

## left_orbit — priority 200, mansion mode (window `led_mansion_2`)

Started by the **center scoop** (`start_events: start_mode_orbit`). Replaces the old
`ghost_ship`.
- **counter `left_orbit_counter`:** counts `shot_left_orbit_hit`, completes at **2**,
  posts `left_orbit_complete` (+3000) = DONE/reward.
- Left-orbit hits +1000, blink `led_advance`; solid (`led_advance: white`) on complete.
- **Start callout:** `taf_start_orbit` (straight to the vault) 2s after start.
- **stop_events:** `left_orbit_complete, ball_will_end`.

## mansion — priority 1000, starts `ball_starting`, stops `ball_will_end`

The orchestrator (was `ship_modes`). Owns the next-mode pointer (`next_mode`), the
MARKED/DONE window state, mansion-LED blink/solid, and slingshot rotation. Full flow:
`docs/architecture.md`. Mode-local shows in `modes/mansion/shows/`.

> The "nothing left" pointer sentinel is the string **`idle`**, never `none` (MPF
> coerces `none`/`null` to null and `variable_player` rejects it).

## center_scoop_lit — priority 1002, starts `ball_starting`, stops `ball_will_end`

The scoop arbiter. Runs the whole ball; tracks `scoop_plugged`.
- Plugged + a window pending → scoop hit posts `start_mode_grave` / `start_mode_orbit`
  (marks the window, starts the mode, **unplugs** the scoop).
- Unplugged → scoop hit plays `taf_not_plugged_in`.
- Bear ramp (`s_right_ramp_entry`) while unplugged → `scoop_replugged` re-lights it.
- `led_center_scoop` red when plugged, off when unplugged.

## test_mode — empty

`modes/test_mode/` has no config and is not registered in `config.yaml`. Inert.
