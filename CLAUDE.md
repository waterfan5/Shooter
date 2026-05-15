# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

Open `index.html` directly in a browser — no build step, no server, no dependencies.

```
start index.html          # Windows
open index.html           # macOS
```

After any edit, refresh the browser tab. There are no tests or linters configured.

## Repository layout

Everything lives in a single file: `index.html` (HTML + CSS + JS, ~450 lines). The GitHub remote is `https://github.com/waterfan5/Shooter`. Commit and push after every meaningful change.

## Architecture

The game is a vanilla-JS Canvas game using `requestAnimationFrame`. All state is module-level (`let` variables). There are no classes, no modules, no bundler.

### Game loop (`loop` → `update` + `draw`)

`loop(timestamp)` is the RAF callback. It computes `dt` (delta-time in seconds, capped at 50 ms), ticks the `explosions` array, advances the `levelClear` timer, then calls `update(dt)` and `draw()`.

`update(dt)` handles all mutation: player movement, power-up countdown, bonus ship movement, bullet movement + collision (aliens and bonus ship), alien group movement + wall-bounce + drop, and win/lose detection.

`draw()` is purely visual and reads state — it never mutates game variables.

### State machine

`state` is a string with four values:

| Value | Description |
|-------|-------------|
| `'start'` | Title screen |
| `'playing'` | Main game loop active |
| `'levelClear'` | 2.2 s overlay; `loop` advances level then returns to `'playing'` |
| `'gameOver'` | Score screen |

`startGame()` resets score/level/powerUpTimer and calls `initLevel()`. `initLevel()` resets everything except `powerUpTimer` (which intentionally persists across levels).

### Alien group movement

Aliens are stored as a flat array of `{row, col, alive, pts}`. There are no per-alien positions — positions are computed on-the-fly with `alienX(col)` / `alienY(row)` from two group offsets: `alienOffsetX` (horizontal, updated every frame) and `alienGroupY` (vertical, incremented by `DROP_STEP` on each wall bounce).

Effective speed scales with kills: `baseAlienSpeed + (killCount / totalAliens) * SPEED_BOOST`. `baseAlienSpeed` itself increases by 12 px/s per level.

### Bullets

`bullets` is an array of `{x, y, vx, vy, dead?}`. Normal fire adds one bullet (vx=0); triple-shot adds three (straight up, ±`POWERUP_ANGLE` = 10°). Bullets are filtered after each `update` pass using the `dead` flag. Only one volley can be in flight at a time (`fireBullet` returns early if `bullets.length > 0`).

### Bonus ship & power-up

A bonus ship spawns every 10 kills (`nextBonusAt` tracks the threshold). It drifts horizontally at `BONUS_SPEED` px/s just below the HUD. Hitting it sets `powerUpTimer = 10` (seconds). While `powerUpTimer > 0`, `fireBullet` emits three angled bullets and the player ship renders purple.

### Key constants to tune gameplay

| Constant | Default | Effect |
|----------|---------|--------|
| `BASE_SPEED` | 10 | Alien px/s at level 1 with all aliens alive |
| `SPEED_BOOST` | 240 | Extra px/s added as aliens are killed |
| `DROP_STEP` | 20 | Pixels aliens descend per wall bounce |
| `BONUS_SPEED` | 55 | Bonus saucer horizontal speed |
| `POWERUP_DURATION` | 10 | Triple-shot duration in seconds |
| `POWERUP_ANGLE` | 10° | Spread angle for side bullets |
