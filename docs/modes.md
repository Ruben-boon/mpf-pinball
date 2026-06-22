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

## Default rules — always-active scoring modes (priority 500)

A set of independent rule modes that all run the **whole ball** (`start_events:
ball_starting`, `stop_events: ball_will_end`) and score by default. They are
**suppressed** during multiball / focus modes via a player-var flag rather than by
stopping the modes (stopping would destroy their state — see gotchas).

**Suppression contract** (confirmed working):
- `base` owns the flag `current_player.default_rules_off` (0 = rules live, 1 = suppressed),
  reset to 0 each ball.
- Posting `disable_default_modes` sets it to 1; `enable_default_modes` sets it to 0.
  Both handlers live in `base` (always-on).
- Every scoring/spotting event in the rule modes is guarded with
  `{not current_player.default_rules_off ...}`.
- A focus mode posts `disable_default_modes` on start and `enable_default_modes` on
  **`mode_<focus>_will_stop`** (see gotcha #2). `left_orbit` is the live test harness:
  disables on start, ends on either the 2-orbit objective (`left_orbit_objective_met`)
  or a 30s fallback timer (`left_orbit_timer_done`), and re-enables on
  `mode_left_orbit_will_stop`. Confirmed: the rules pause on start and resume on end.

The modes:
- **million_ramp** — left/side ramp (`shot_left_ramp_hit`). +1M per hit, 1M→10M cap,
  resets each ball (`million_ramp_value`). Also spots the first unlit GRAVE letter
  (handled in grave_letters). Ramp-loop SFX is played by `base`, not here.
- **bear_kick** — center/main/right ramp (`shot_right_ramp_hit`). +1 Bear Kick (+2 when
  `bear_kick_lit`, set by the near-left inlane — TODO, no switch yet). Posts
  `chair_light_request`. This shot is also GRAVE "V".
- **graveyard_jets** — pops `s_pop1..5`. Owns **Graveyard Value** (`graveyard_value`,
  ball-scoped, starts 1M, caps 4M): +20K per lit jet (GRAVE incomplete), +10K unlit
  (GRAVE complete). Each hit posts `advance_flashing_room`.
- **swamp** — `s_swamp_feed`. Scores `graveyard_value` ×1, or ×5 when `swamp_lit`
  (**lit by default for now** — TODO: should be armed by the small-flipper vault pass,
  no switch yet). A swamp shot only counts if the previous tracked shot was **not**
  `s_vault_subway` (tracked via `last_swamp_shot`).
- **swamp_kickout** — `s_right_scoop` + `c_right_scoop` kick. Only counts if the previous
  tracked shot was **not** `s_swamp_feed`. Posts `spot_grave_e` (GRAVE "E") and, when the
  chair is yellow-lit, `award_flashing_room`. **The kick always fires**, even suppressed.
- **grave_letters** — GRAVE tracking. Letters are **plain player vars** `grave_g..grave_e`
  (NOT MPF shots — see gotcha #1), LEDs rendered from those vars via `light_player` on a
  `redraw_grave` event (also posted on `enable_default_modes` so they repaint after a
  focus mode). G/R/A = standups, V = `s_ramp_top`, E = `spot_grave_e`. Left ramp spots the
  first unlit letter. Each newly-lit letter posts `advance_jet`. Completing G-R-A-V-E
  awards the **game-scoped** GRAVE bonus (`grave_bonus_level`, 2M→10M) and resets the
  letters for another lap. `grave_complete` gates the jets' lit state.
- **inlanes** — empty stub (no inlane switches exist yet). Planned: near-left inlane lights
  "2 Bear Kicks"; right inlane temporarily lights the chair.

> **Gotcha #1 — don't suppress state-bearing rules by stopping the mode.** Stopping a mode
> destroys its state and restarting re-runs its start logic (resetting it). And disabling a
> *shot* turns its show OFF and won't restore the lit show on re-enable — the GRAVE LEDs went
> dark and couldn't relight. Fix: keep the mode running, store state in player vars, render
> LEDs from those vars, and gate scoring with the `default_rules_off` flag.
>
> **Gotcha #2 — re-enable on `_will_stop`, not `_stopped`.** A mode's own `event_player` does
> **not** run on `mode_<x>_stopped` (the mode is already gone), so posting `enable_default_modes`
> there silently never fires and the rules stay dead. Post it on `mode_<x>_will_stop` (mode still
> active) — or from an always-on mode like `base`.

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
