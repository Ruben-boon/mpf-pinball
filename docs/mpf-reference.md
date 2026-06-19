# MPF reference links

Curated links into the **official MPF documentation** for the features this project
actually uses. Use these instead of answering from memory.

## Version caveat (important)

This machine runs **MPF 0.57.3 (the 0.57 LTS line)**. The docs site's "latest" tracks
**0.80**, which has diverged in places. The links below use `missionpinball.org/latest/…`
because those paths resolve; when a behavior matters, confirm it holds for **0.57**, and
prefer the 0.57 LTS docs if a version selector is offered. If something in the live docs
contradicts how this repo's config behaves on 0.57, trust the repo's observed behavior and
flag the discrepancy.

- Docs home: https://missionpinball.org/latest/
- User forum: https://groups.google.com/g/mpf-users

URL patterns observed:
- Config-section reference pages: `https://missionpinball.org/latest/config/<name>/`
- Conceptual guides: `https://missionpinball.org/latest/game_logic/<topic>/`

## Core concepts

- Modes: https://missionpinball.org/latest/game_logic/modes/
- Events (the event system): https://missionpinball.org/latest/events/
- Config file format / `config_version`: https://missionpinball.org/latest/config/

## Game logic (what this project leans on heavily)

- Shots: https://missionpinball.org/latest/game_logic/shots/
- Sequence shots: https://missionpinball.org/latest/game_logic/shots/ (see "Sequence Shots")
- Shot profiles: https://missionpinball.org/latest/config/shot_profiles/
- Shot groups: https://missionpinball.org/latest/config/shot_groups/
- Counters (logic blocks): https://missionpinball.org/latest/game_logic/logic_blocks/counters/
- Achievements: https://missionpinball.org/latest/game_logic/achievements/

## Config "players"

- event_player: https://missionpinball.org/latest/config/event_player/
- variable_player: https://missionpinball.org/latest/config/variable_player/
- show_player: https://missionpinball.org/latest/config/show_player/
- light_player: https://missionpinball.org/latest/config/light_player/
- sound_player: https://missionpinball.org/latest/config/sound_player/
- slide_player: https://missionpinball.org/latest/config/slide_player/

## Hardware

- OPP platform: https://missionpinball.org/latest/hardware/opp/
- Switches: https://missionpinball.org/latest/config/switches/
- Coils / drivers: https://missionpinball.org/latest/config/coils/
- Lights / LEDs: https://missionpinball.org/latest/config/lights/
- Flippers: https://missionpinball.org/latest/game_logic/mechs/flippers/
- Autofire coils: https://missionpinball.org/latest/config/autofire_coils/
- Ball devices: https://missionpinball.org/latest/game_logic/mechs/ball_devices/

## Shows, display, sound

- Shows (`#show_version=6`): https://missionpinball.org/latest/shows/
- Displays: https://missionpinball.org/latest/config/displays/
- Slides & widgets: https://missionpinball.org/latest/config/slides/
- DMD / color_dmd effect: https://missionpinball.org/latest/config/displays/
- Sound system / tracks: https://missionpinball.org/latest/sound/

> If a link 404s for the installed version, drop to the docs home and use its search /
> version selector rather than guessing a path. Verify a page before citing exact syntax.
