# Audio (sound & music)

How sound is wired in this machine. Only **confirmed-working** facts go here — an entry
is added once the change has been tested (`mpf both -X -vV`, clean logs) **and** the user
has confirmed it works on the machine. If it isn't confirmed, it doesn't belong here yet.

## Tracks (`config/config.yaml` → `sound_system.tracks`)

| Track | simultaneous_sounds | volume | Used for |
|---|---|---|---|
| `music` | 1 | 0.5 | Background music (one at a time — a new music sound replaces the current) |
| `voice` | 1 | 0.7 | Callouts (Gomez/Morticia/Thing lines) — layers *over* music |
| `sfx` | 8 | 0.4 | Hit/score effects |

Because `music` is `simultaneous_sounds: 1`, starting a new music sound stops the previous
one automatically. `voice` and `sfx` are on separate tracks, so callouts and effects play
**on top of** the music without interrupting it.

## Sound assets

- Sound definitions live in `config/sounds.yaml`. Each entry has `file:` + `track:`.
- MPF auto-discovers every audio file under `sounds/` recursively. The **track is assigned
  by folder name** via `assets.sounds` in `config/config.yaml` (`music/`, `sfx/`, `voice/`,
  and `taf_remastered/` → default `sfx`). A loose file in a folder with no track mapping
  makes MPF crash on load: `AssertionError: Sound X does not have a valid track`.
- `sounds/taf_remastered/` is the TAF (Addams Family) remastered altsound pack (~290 files).
  `altsound.csv` indexes them; `CHANNEL 0` = looping music, `CHANNEL 1` = jingles, blank = sfx/voice.

## Confirmed-working setup

### Music (CONFIRMED working)
| Slot | Event | Sound def | File |
|---|---|---|---|
| Attract loop | `mode_attract_started` (loops -1) | `taf_attract` | `0x0018-addams_theme.wav` |
| Prelaunch (ball waiting) | `mode_base_started` | `taf_prelaunch` | `0x0006-wait_1.wav` |
| Main gameplay (ball in play) | first `playfield_active` per ball | `gameplay_themes` playlist | main_theme 1–6 |

Flow per ball: `taf_prelaunch` plays on `mode_base_started`; on the first `playfield_active`
it stops (1s fade) and the **gameplay theme rotation** starts.

#### Gameplay theme rotation (playlist)
- Dedicated **`music_playlist`** track (`type: playlist`, `crossfade_time: 2s`) in
  `config/config.yaml` — playlists require their own playlist-type track, separate from
  `music`. Theme sounds `taf_theme_1..6` are defined on this track.
- **`gameplay_themes`** playlist (`config/sounds.yaml`): `repeat: True` loops the whole list
  forever; each theme crossfades (2s) into the next; after theme 6 it loops back to 1.
- Started by `playlist_player` on the one-shot **`start_gameplay_music`** event. Under the
  event key, the nesting is `<track_name>: { playlist: <playlist_name>, action: play }` —
  the key is the TRACK name (`music_playlist`), NOT the playlist name. Stopped on `ball_will_end`.

##### Start it ONCE per ball (the overlap fix)
`playfield_active` fires every time the ball returns to the playfield (after a scoop/lane
eject). Triggering the playlist directly on `playfield_active` restarts/stacks it and causes
overlapping music. Fix: a `themes_started` player-var flag (set 0 on `mode_base_started`),
and base mode's `event_player` posts the one-shot `start_gameplay_music` only on the FIRST
playfield_active: `playfield_active{not current_player.themes_started}: start_gameplay_music`.
NOTE: don't set the flag with `variable_player` on the same `playfield_active` event and then
gate the playlist on it — config-player ordering makes the flag flip before the playlist
condition is checked (race), so the playlist never starts. Route through a one-shot event
instead.

### Attract → game interrupt (CONFIRMED working)
- Attract music does **not** stop on its own. It is explicitly stopped on
  `mode_attract_will_stop` (0.3s fade) in `modes/attract/config/attract.yaml`. This is what
  makes the music cut out the moment a game starts.

### Welcome callout (CONFIRMED working)
- On `mode_base_started`, the `taf_welcome` **sound_pool** plays on the voice track, layered
  over the music. Pool randomly alternates between `taf_welcome_back` (`0x7a18`) and
  `taf_welcome_guest` (`0x7a29`). Defined in `config/sounds.yaml` under `sound_pools:`.

### Scoring sfx — distinct pools per type (CONFIRMED working)
No sound is shared across element types. Each type has its own pool/sound:

| Type | Sound (`config/sounds.yaml`) |
|---|---|
| Pop bumpers | `taf_pop` — random ha/ho/hu (`taf_haho_1..11`) |
| Slingshots | left=`taf_rebound_3`, right=`taf_rebound_4` (distinct, no pool) |
| Grave targets | `taf_target` — ting 1–4 |
| Base right scoop | `taf_scoop` — combo 1–3 |
| Ramps (base/ramp_alt/left_ramp) | `taf_ramp` — rebound 1–2 |
| Orbit (ghost_ship) | `taf_orbit` — whistle 1–3 |
| IT hole | `taf_thing_fx` |
| Mode completes | distinct per mode: grave=`spirit_fx`, ramp_alt=`matchup`, left_ramp=`stairs`, ghost=`mansion_2` |
| Ship collect 20k | `taf_jackpot` |

- The old `sword_slice` references (grave/ramp_alt/left_ramp/ghost_ship) are gone — they were
  undefined and silent. grave_targets sounds were also on the wrong event (`*_unlit_hit`);
  moved to the `*_lit_hit` events that the scoring/shows actually use.

### Center scoop (CONFIRMED working)
- On capture (`s_center_scoop_active`, played from base mode), `taf_mansion_1` (`0x00d1`)
  plays and **ducks the music** until the callout finishes — see ducking below.
- `bd_center_scoop` holds the ball before ejecting via
  `eject_events: s_center_scoop_active:2s` in `config/devices.yaml`. NOTE: the 2s hold is a
  real-hardware behavior; in smart-virtual (`-X`) the "unexpected ball" path ejects after the
  ~0.5s virtual confirm and ignores the delay, so verify the hold length on the machine.

## Ducking (pause music under a callout)

To make a sound duck/pause a track while it plays, add a `ducking:` block to the sound def
(`config/sounds.yaml`). Example (`taf_mansion_1`, ~2.5s clip, ducks the `music` track):

```yaml
ducking:
  target: music        # track to attenuate
  delay: 0
  attack: 0.1sec       # how fast the duck ramps in
  attenuation: -30db   # how much to drop the target (large = near-pause)
  release_point: 0.3sec  # start releasing this far before the sound ends
  release: 0.4sec      # how fast the target fades back
```

## Gotchas (learned the hard way)

- **A sound played from a mode that is stopping gets cut off.** Game-start callouts must be
  played from a long-lived mode (`base`, on `mode_base_started`), **not** from `attract` on
  `mode_attract_will_stop` — attract tears down its sound context immediately and kills the
  just-started sound before it's audible. Event order at game start:
  `mode_game_will_start` → `mode_attract_will_stop` → `mode_attract_stopped` →
  `mode_game_started` → `game_started` → `ball_will_start` → `ball_starting` →
  `mode_base_started`.
- A `sound_player` entry for `game_started` placed in **attract** mode never fires, because
  attract is already stopped by the time `game_started` arrives.
- `sword_slice` is referenced by several modes (grave_targets, ramp_alternate, left_ramp,
  ghost_ship) but is **not defined** in `sounds.yaml` → those hits are currently silent.
  (To be fixed in the sfx phase.)
