# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. The README and all user-facing UI strings are in Spanish; keep new UI text in Spanish.

## Running

```bash
python3 -m http.server 8000   # then open http://localhost:8000
# or simply: xdg-open index.html
```

There are no automated tests — verify changes by playing the game in a browser.

## Architecture

Three files: `index.html` (DOM + two canvases + overlay), `style.css` (dark theme), and `game.js`, which holds all logic as top-level functions sharing module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of it and is also the restart handler.

Key conventions in `game.js`:

- **Cell values double as color/type indices.** `board` is a `ROWS × COLS` matrix of `0` (empty) or `1–7`; each shape in `PIECES[type]` is filled with its own type number, so `merge()` copies shape values straight into the board and `drawBlock()` looks them up in `COLORS`. `PIECES`, `COLORS` and the type index must stay aligned (index 0 is `null`).
- **`collide(shape, ox, oy)` is the single source of truth** for movement, rotation (`tryRotate` with kicks `[0, -1, 1, -2, 2]`), ghost projection (`ghostY`), gravity, and game-over detection in `spawn()`. Cells with `ny < 0` are allowed (above the board).
- **Piece lifecycle:** `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()` (promotes `next` to `current`, generates a new `next`, calls `endGame()` if the new piece collides immediately).
- **Game loop:** `loop()` uses `requestAnimationFrame`, accumulating `dt` into `dropAccum` and applying gravity when it exceeds `dropInterval`. Pause/game-over stop it via `cancelAnimationFrame(animId)`; `loop` itself does not check `paused`/`gameOver`. Note that when game over is triggered from inside `loop` (gravity lock), `loop` still schedules a new frame after `endGame()` has cancelled the old id.
- **Rendering** is a full redraw every frame (`draw()`: grid → board → ghost at alpha 0.2 → current piece). The next-piece preview is only redrawn on `spawn()` (`drawNext()`, 4×4 cells of 30px).

## Coupled values

Canvas sizes in `index.html` are hardcoded and must match `game.js`: `#board` is `COLS × BLOCK` by `ROWS × BLOCK` (300×600), and `#next-canvas` is 4 × 30 = 120px square. Changing `COLS`, `ROWS`, or `BLOCK` requires updating the HTML. DOM element IDs used by `game.js`: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`.
