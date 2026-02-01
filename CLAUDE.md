# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Mission Pinball Framework (MPF)** project for a physical pinball machine. MPF is a Python-based framework that runs pinball machines, handling hardware control, game logic, modes, scoring, displays, sounds, and LED shows through YAML configuration files.

**Hardware Platform**: OPP (Open Pinball Project) Gen2 on COM3/COM4
**MPF Config Version**: 6

## Essential Commands

### Running the Game
```bash
# Run game + media controller (normal operation)
mpf both

# Run with verbose logging (for debugging)
mpf both -vV

# Run in smart virtual mode (no hardware required for testing)
mpf both -X -vV

# Run game only (no media controller/display)
mpf game -b -vV
```

### Testing & Validation
```bash
# Always test config changes in virtual mode first
mpf both -X -vV

# Check recent logs for errors
tail -100 /Users/ruben/Pinball/mpf-pinball/logs/*.log
```

## Architecture Overview

### Configuration Hierarchy

MPF uses a hierarchical configuration system where files are loaded in order:

1. **Machine-wide configs** (`config/config.yaml` is the entry point):
   - `hardware.yaml` - OPP platform configuration, PSU settings
   - `switches.yaml` - All switches (flippers, slingshots, targets, scoops, pop bumpers)
   - `coils.yaml` - All coils with pulse/hold settings
   - `lights.yaml` - LED definitions and channels
   - `devices.yaml` - Ball devices (trough, scoops), flippers, autofire coils
   - `displays.yaml`, `slides.yaml` - Display configuration
   - `sounds.yaml`, `images.yaml`, `videos.yaml` - Asset definitions

2. **Mode-specific configs** (`modes/[mode_name]/config/[mode_name].yaml`):
   - Each mode has its own directory with `config/` and `shows/` subdirectories
   - Modes define game logic, scoring, shots, counters, timers, and events

### Core Pinball Concepts

**Modes**: Game states that can start/stop and run concurrently with priorities
- `base` (priority 250): Main game mode with shots, scoring, GI lighting, music
- `attract` (priority N/A): Attract mode with light shows and music when idle
- `pop_bumpers` (priority 200): Tracks 20 pop bumper hits to complete
- `grave_targets` - Target completion mode
- `ramp_alternate` - Ramp shot progression
- `ghost_ship`, `left_ramp_mode` - Additional game modes
- `ship_modes` (priority 1000): Meta-mode tracking completion of other modes via achievements
- `center_scoop_lit` - Scoop lighting and collection

**Mode Lifecycle**:
- Modes start on specific events (e.g., `ball_starting`, custom completion events)
- Modes stop on events (e.g., `ball_will_end`, mode completion)
- Higher priority modes can override lower priority modes

**Key MPF Devices**:
- **Shots**: Define switch sequences (e.g., ramps require entry switch → top switch within 3s)
- **Counters**: Track hits and trigger events at thresholds
- **Achievements**: Track mode completion states (not started → started → completed)
- **Ball Devices**: Physical ball storage (trough, scoops) with eject coils
- **Shows**: Timed sequences of light/LED states, synchronized with sounds

### Hardware Naming Conventions
- Switches: `s_[name]` (e.g., `s_left_flipper`, `s_pop1`, `s_target_g`)
- Coils: `c_[name]` (e.g., `c_flipper_left_main`, `c_trough_eject`)
- LEDs: `led_[name]` (e.g., `led_mansion_1`, `led_gy`)
- Shows: Descriptive names (e.g., `blue_whirlpool`, `show_blink`)

### Sound System Architecture

Three audio tracks with different purposes:
- **music** (1 simultaneous sound, 50% volume): Background music, loops
- **voice** (1 simultaneous sound, 70% volume): Callouts and announcements
- **sfx** (8 simultaneous sounds, 40% volume): Sound effects (slingshots, bumpers, etc.)

Sounds are organized in `sounds/` with subdirectories:
- `music/` - Background tracks
- `sfx/` - Sound effects
- `voice/` - Callouts

### Event System

MPF is event-driven. Everything communicates through events:
- Switch activations generate events (`s_pop1_active`)
- Shots completed generate events (`shot_right_ramp_hit`)
- Modes post events (`mode_attract_started`, `pop_mode_complete`)
- Counters/timers post events (`pop_bumpers_20_hits_reached`)
- Events trigger actions via `event_player`, `show_player`, `sound_player`, `variable_player`

**Achievement System Pattern** (used in `ship_modes`):
- Individual modes complete and post events (`pop_mode_complete`)
- Ship_modes listens for completions and starts achievements (`ship_pop_mode_ready`)
- Achievements blink LEDs (`show_blink` on `led_mansion_1-5`)
- Collecting at center scoop (`ship_pop_mode_collected`) completes the achievement
- Completing achievement lights LED solid and posts completion event

### Display System

Base mode creates the main game slide with:
- Score display (large centered text with Metal Mania font)
- Player number (bottom left)
- Ball number (bottom right)
- Background image

Widget types: text, image (positioned with x/y, anchored)

### Project Structure

```
mpf-pinball/
├── config/               # Machine-wide configuration
│   ├── config.yaml       # Main entry point, loads other configs
│   ├── hardware.yaml     # OPP platform settings
│   ├── switches.yaml     # All switch definitions
│   ├── coils.yaml        # All coil definitions
│   ├── lights.yaml       # LED channel mappings
│   ├── devices.yaml      # Flippers, ball devices, autofire coils
│   ├── displays.yaml     # Display settings
│   ├── slides.yaml       # Slide definitions
│   └── sounds.yaml       # Sound asset definitions
├── modes/                # Game modes
│   ├── attract/          # Attract mode (light shows, music)
│   ├── base/             # Main game mode (always running during play)
│   ├── pop_bumpers/      # Pop bumper completion mode
│   ├── grave_targets/    # Target completion mode
│   ├── ship_modes/       # Meta-mode tracking mode completions
│   └── [other_modes]/
│       ├── config/       # Mode YAML configuration
│       └── shows/        # Mode-specific light shows
├── shows/                # Global light shows (attract sequences)
├── sounds/               # Audio files (music/, sfx/, voice/)
├── data/                 # Runtime data (machine_vars.yaml, audits.yaml)
├── monitor/              # Monitor app config
└── .claude/agents/       # 9 specialized MPF development agents
```

## Working with This Codebase

### Configuration Files
- All configs use `#config_version=6` header
- YAML syntax: 2-space indentation, no tabs
- Hardware numbers use format: `[board]-[wing]-[channel]` (e.g., `0-0-19`, `1-0-2`)

### Making Changes

1. **Always edit existing configs** rather than creating new ones
2. **Test in virtual mode** (`mpf both -X -vV`) before hardware testing
3. **Check logs** after config changes for errors/warnings
4. **Verify hardware addresses** match physical machine wiring
5. **Use specialized agents**: This project has 9 Claude Code agents for different MPF tasks (see `.claude/agents/README.md`)

### Specialized Agents Available

- **mpf-config-tester**: Test configs and create hardware test plans
- **mpf-config-expert**: Hardware configuration (switches, coils, LEDs)
- **mpf-mode-builder**: Create game modes with shots, timers, logic
- **mpf-show-creator**: LED shows and light animations
- **mpf-sound-designer**: Audio tracks, ducking, sound pools
- **mpf-scoring-logic**: Scoring, combos, multipliers, player variables
- **mpf-multiball-wizard**: Multiball modes, ball locks, jackpots
- **mpf-display-designer**: Slides, widgets, animations
- **mpf-hardware-debugger**: Troubleshoot hardware issues

### Common Development Workflow

1. Identify the change needed (new mode, hardware addition, scoring change)
2. Edit appropriate config file(s) in `config/` or `modes/[mode]/config/`
3. Run `mpf both -X -vV` to test configuration in virtual mode
4. Check logs for errors: `tail -100 logs/*.log`
5. If no errors, test on physical hardware with `mpf both -vV`
6. Iterate based on behavior

### Mode Development Pattern

When creating a new mode:
1. Create directory: `modes/[mode_name]/`
2. Create subdirectories: `config/` and `shows/`
3. Create mode config: `modes/[mode_name]/config/[mode_name].yaml`
4. Define mode metadata (start_events, stop_events, priority)
5. Add game logic (shots, counters, timers, logic_blocks)
6. Add scoring via `variable_player`
7. Add light shows via `show_player`
8. Add sounds via `sound_player`
9. Register mode in main `config/config.yaml` under `modes:`

### Current Game Progression

The game uses a "ship modes" progression system:
- Players complete individual modes (pop_bumpers, grave_targets, ramp_alternate, ghost_ship, left_ramp_mode)
- Each completion lights an achievement in ship_modes
- Achievements blink their assigned mansion LED
- Collecting achievements at center scoop completes them
- Completed achievements light LEDs solid

## Important Notes

- **Hardware safety**: Coils have `default_pulse_ms` (typically 12-25ms) and power settings to prevent damage
- **Ball device configuration**: Ball devices need switches and eject coils, with timeouts for safety
- **Flippers**: Have separate main (pulse) and hold coils, enabled on `machine_reset_phase_3`
- **Autofire coils**: Slingshots and pop bumpers fire automatically when switches activate
- **Persistent data**: Machine variables and audits stored in `data/`
- **Sound asset loading**: Sounds use `load: on_demand` by default to save memory

## Resources

- **MPF Documentation**: https://missionpinball.org/latest
- **MPF Forum**: https://groups.google.com/g/mpf-users
- **Agent Documentation**: `.claude/agents/README.md`
- **Agent Setup Guide**: `MPF_CLAUDE_AGENTS_SETUP.md`
