# American Football Browser Game

## Workflow Rule
**Always commit and push after every edit.** No exceptions.
```bash
git add football.html && git commit -m "Description" && git push
```

---

## Project Overview
A single-file browser-playable American football game built with vanilla HTML5 Canvas + JavaScript. No build tools, no dependencies — open `football.html` directly in any browser.

## File
- `football.html` — the entire game (HTML + CSS + JS, ~600 lines)

## Controls
| Key | Action |
|---|---|
| WASD | Move QB (pre-snap / pocket / scramble) or move ball carrier (post-catch) |
| Space | Hike the ball |
| 1–5 | Throw to receiver #N (only valid post-snap; disabled once QB crosses LOS) |
| P | Punt (4th down only, pre-snap) |

## Game States
```
PRE_SNAP → PLAY_ACTIVE → BALL_IN_AIR → RECEIVER_RUNNING → Tackle / Touchdown
                ↓ (QB crosses LOS)
           QB_SCRAMBLE (still PLAY_ACTIVE, qbRunning=true)
                ↓
           Tackle or Touchdown
         ↓ (P on 4th)
         PUNT → DEFENSE_SIM → PRE_SNAP
```

## Entities
| Entity | Count | Color | Notes |
|---|---|---|---|
| Quarterback | 1 | Yellow helmet / yellow jersey | WASD; can scramble past LOS |
| Receivers | 5 | Green helmet / green jersey | Numbered 1–5; run predefined routes |
| Defenders | 5 | Red helmet / dark red jersey | Dynamic strategy per play (see below) |
| Ball | 1 | Brown oval | Attached to carrier or animated in-flight |

### Defender Positions
- **2 Defensive Ends** — just past LOS on defensive side
- **1 Middle Linebacker** — ~8 yards behind LOS
- **2 Cornerbacks/Safeties** — deep defensive backfield (~17 yards back)

### Defensive Strategy (randomized per play at snap)
Each defender independently rolls on snap:
- **40%** → rushes the QB
- **60%** → covers the nearest receiver

When QB scrambles past LOS, **all 5 defenders** immediately converge on the QB regardless of assigned strategy.

## Gameplay Rules
- **Downs**: 4 downs to gain 10 yards; first down resets the series
- **Sack**: QB holds ball ~3s in pocket without throwing → sack (3–10 yd loss)
- **QB Scramble**: QB crosses LOS → `qbRunning = true`; passing disabled; all defenders chase; touchdown if QB reaches opponent endzone
- **Interception**: Defenders near receiver at catch time have a proximity-based % chance to intercept
- **Punt**: 4th down only; ball placed ~40 yds forward (±10 random)
- **Scoring**: Touchdown = +7 (player or opponent); no PAT or FG for player
- **Defense sim** (opponent possession): 60% 3-and-out punt back, 25% FG attempt (40% success = +3), 15% TD (+7)
- **Clock**: 4 quarters × 2 minutes real time; game ends at Q4 0:00

## Field Layout
- Canvas: 800×620px; top 60px = HUD
- Field: 100 yards, ~5.6px/yard; own endzone at bottom (high y), opponent at top (low y)
- Yellow dashed line = line of scrimmage; blue dashed = first-down marker
- Yard lines every 10 yards; hash marks every 5 yards
- Touchdown triggers when `yToYard(carrier.y) >= 90` (entering opponent endzone)

## Architecture Notes
- Pure state machine: `state` variable + `qbRunning` boolean drive all logic
- `yardToY(yard)` → canvas y; `yToYard(y)` → yard number (higher yard = lower y)
- Own endzone = bottom (high y); opponent endzone = top (low y)
- `receivers[i].route` = array of {x,y} waypoints traversed in order
- Ball flight: `ballAir.cx/cy` recalculated each frame using receiver's *live* position so ball always lands on the receiver
- `def.strategy` assigned fresh every snap in `startPlay()`; replaces static `role`
- `updateDefSim` ticks a countdown then calls `resolveDefenseSim()` for random outcome

## Player Rendering (`drawPlayer`)
Each player is a layered canvas drawing (drawn bottom to top):
1. Drop shadow (ellipse)
2. Legs + cleats
3. Jersey body (trapezoid)
4. Jersey number / label (11px bold white, centered on chest)
5. Shoulder pad bumps
6. Arms + gloved hands
7. Tucked football (if `hasBall`)
8. Helmet + stripe
9. Facemask bars
10. Neck

## Development
No build step needed. Edit `football.html`, refresh browser, then **commit and push**.
