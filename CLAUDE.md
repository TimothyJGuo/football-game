# American Football Browser Game

## Project Overview
A single-file browser-playable American football game built with vanilla HTML5 Canvas + JavaScript. No build tools, no dependencies — open `football.html` directly in any browser.

## File
- `football.html` — the entire game (HTML + CSS + JS, ~500 lines)

## Controls
| Key | Action |
|---|---|
| WASD | Move QB (pre-snap / pocket) or move ball carrier (post-catch) |
| Space | Hike the ball |
| 1–5 | Throw to receiver #N (only valid post-snap while QB holds ball) |
| P | Punt (4th down only, pre-snap) |

## Game States
```
PRE_SNAP → PLAY_ACTIVE → BALL_IN_AIR → RECEIVER_RUNNING → Tackle / Touchdown
         ↓ (P on 4th)
         PUNT → DEFENSE_SIM → PRE_SNAP
```

## Entities
| Entity | Count | Color | Notes |
|---|---|---|---|
| Quarterback | 1 | Yellow | WASD controlled pre-snap + in pocket |
| Receivers | 5 | Green | Numbered 1–5; run predefined routes |
| Defenders | 4 | Red | 2 rush QB, 2 cover receivers (AI) |
| Ball | 1 | Brown oval | Animates in-flight with arc |

## Gameplay Rules
- **Downs**: 4 downs to gain 10 yards; first down resets series
- **Sack**: QB holds ball ~3s without throwing → sack (random 3–10 yd loss)
- **Interception**: Defenders near receiver have % chance to pick the ball
- **Punt**: 4th down only; ball placed ~40 yds forward (±10 random)
- **Scoring**: Touchdown = +7 (player or opponent)
- **Defense sim** (opponent possession): 60% 3-and-out, 25% FG attempt (40% success = +3), 15% TD (+7)
- **Clock**: 4 quarters × 2 minutes real time; game ends at Q4 0:00

## Field Layout
- Canvas: 800×620px; top 60px = HUD
- Field: 100 yards, ~5.6px/yard; own endzone at bottom, opponent at top
- Yellow dashed line = line of scrimmage; blue dashed = first-down marker
- Yard lines every 10 yards; hash marks every 5 yards

## Architecture Notes
- Pure state machine: `state` variable drives all update/render logic
- `yardToY(yard)` / `yToYard(y)` convert between game units and canvas pixels
- `receivers[i].route` = array of waypoints the receiver runs toward
- `ballAir` object tracks in-flight ball position via parametric interpolation + arc
- `updateDefSim` ticks a countdown then calls `resolveDefenseSim()` for random outcome

## Development
No build step needed. Edit `football.html` and refresh the browser.
