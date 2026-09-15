# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-page Tetris clone in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS — no dependencies, no `package.json`, no build step or bundler. This is a standalone project within a larger Udemy course workspace; see the parent `CLAUDE.md` for how this folder relates to sibling lesson folders.

## Running the game

No install or build step. Either open `index.html` directly, or serve it over a static server (needed if the browser restricts local file access for canvas/scripts):

```bash
python3 -m http.server 8000
# or
npx serve .
```

Then visit `http://localhost:8000`. There are no tests, linter, or build/watch commands in this project.

## Architecture

Three files cooperate with no module system — everything is loaded via a single `<script src="game.js">` and relies on global scope and DOM ids matching between `index.html` and `game.js`:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600, the play field) and `<canvas id="next-canvas">` (next-piece preview), plus HUD elements (`#score`, `#lines`, `#level`) and a shared `#overlay` used for both Pause and Game Over states.
- **`style.css`** — dark/retro arcade visual theme only; no layout logic depends on it.
- **`game.js`** — all game logic, in one file:
  - **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–8` identifying a locked piece.
  - **Pieces**: the 7 classic tetrominoes plus one special 3×3 "tuerca" (nut) piece with an inert hole in its center, all as square matrices in `PIECES`. Any `0` cell in a piece's matrix (not just its bounding-box padding) is inherently a no-op for `collide`/`merge`/`draw` — this is what lets the nut's center hole be non-blocking (other pieces can poke through it once locked) and non-filling (`merge` never writes that cell to `board`, so a completed row through the hole won't clear until something else fills it) with zero special-casing. Rotation is `rotateCW` (transpose + reverse), collision is `collide`, and `tryRotate` implements basic wall kicks by retrying rotation at `x` offsets `[0, -1, 1, -2, 2]`.
  - **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates elapsed time in `dropAccum`, and advances the piece one row once `dropAccum >= dropInterval`.
  - **Locking/scoring**: `lockPiece` → `merge` (writes piece into `board`) → `clearLines` (scans bottom-up, splices full rows, unshifts empty ones at top) → `spawn` (promotes `next` to `current`, generates a new `next`, and calls `endGame` if the new piece immediately collides).
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points per cell dropped, soft drop adds 1 point per row.
  - **Leveling**: level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2` in `draw()`.
  - **Skins**: `drawBlock` is the single integration point for the 4 visual skins (`retro`, `neon`, `pastel`, `pixel`) — it branches on the module-scope `skin` variable to pick fill colors/effects (retro is the unchanged default, neon adds canvas `shadowBlur`/`shadowColor` glow, pastel uses `PASTEL_COLORS` with rounded corners, pixel overlays a deterministic dithering pattern), selected via `#skin-select` and persisted in `localStorage` under `tetris-skin`.
  - State (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) lives in module-level `let` bindings reset by `init()`, which also wires `restartBtn` and starts the loop.

When tuning gameplay, `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, and the initial `dropInterval` are the constants to change at the top of `game.js`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`) since nothing computes canvas size dynamically.
