# Light shows

How shows are stored, and the workflow for authoring new ones — including with the
**showcreator** visual editor.

## Where shows live

- **Global shows:** `shows/*.yaml` — every file opens with `#show_version=6`.
- **Mode-local shows:** `modes/<name>/shows/*.yaml`.
- Shows are triggered with `show_player:` in a mode config, keyed off an event
  (a switch `*_active`, a mode event, etc.). Example in `modes/base/config/base.yaml`:

  ```yaml
  show_player:
    s_target_r_active:
      sweep_left_to_right:
        speed: 40
        loops: 0          # 0 = play once
  ```

## Show YAML format (v6)

Frame list. Each frame is a `- time:` step plus a `lights:` map of `led_* : 'RRGGBB'`.
`time: 0` is the first frame; `time: '+1'` advances one step. A bare `- time: '+1'`
with no `lights:` is a valid "hold / no change" frame.

```yaml
#show_version=6
- time: 0
- time: '+1'
  lights:
    led_left_outlane: 'F9FAF9'
- time: '+1'
  lights:
    led_left_outlane: '000000'
```

`speed:` on the `show_player` entry sets steps/sec; `'000000'` = off.

## Authoring with showcreator (visual editor)

`showcreator` is the official MPF visual LED-show editor (BlitzMax). It exports show
YAML by sweeping image shapes across an LED coordinate map.

**Setup / running (macOS, Apple Silicon):**
- Installed at **`~/Pinball/showcreator`** (from `https://github.com/missionpinball/showcreator`;
  not vendored into this repo).
- Use **`led mac x64.app`** only — it is the x86_64 build and runs under Rosetta 2.
  The other binaries (`led`, `led.app`, `led.debug.app`) are 32-bit i386 and **cannot
  run** on modern macOS.
- It must be launched **from the showcreator dir** so it finds `monitor.yaml`/`shapes/`/`sets/`:

  ```bash
  cd ~/Pinball/showcreator && "./led mac x64.app/Contents/MacOS/led"
  ```
  Launching via `open` fails (cwd becomes `/`, app exits immediately).

**Key facts about this copy:**
- Its `monitor.yaml` is **already mapped to THIS machine's `led_*` names** (mansion,
  letters, swamp, greed, gi_*, etc.) — exported shows reference our real lights.
- A saved **set** (`sets/*.txt`) is the editor's own keyframe format, NOT a show.
  Export the actual show with **`P` + `SHIFT`** ("play set and create script").
- The export is written as **`output<N>.yaml` in the app root** (not into `shows/`),
  and it is emitted as **`#show_version=5`**.

**Cleanup needed on every export** (this old build has two known bugs):
1. Change the header `#show_version=5` → `#show_version=6`.
2. Remove the malformed "no change this frame" blocks — a `- time: '+1'` followed by a
   dangling blank indented line. Replace with a bare `- time: '+1'`.

Then save into `shows/<descriptive_name>.yaml`, wire it via `show_player`, and validate.

## Validate

```bash
mpf both -X -vV     # smart-virtual; reaching "RUNNING" with no ConfigError = show + trigger OK
```
A bad frame or unknown `led_*` name aborts at load, so a clean RUNNING state confirms it.

## Confirmed shows authored this way

- **`shows/sweep_left_to_right.yaml`** — white bar sweeping left→right across the whole
  playfield. Built in showcreator (`leftToRightFull` set), cleaned to v6, mapped to
  `s_target_r_active` in `base`. Tested on hardware, confirmed working (2026-06-22).
