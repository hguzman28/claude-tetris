# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — just static files served or opened directly.

- `index.html` — DOM structure: the `#board` canvas (300×600), the `#next-canvas` preview (120×120), score/lines/level HUD, and the pause/game-over overlay.
- `style.css` — dark/retro arcade visual styling.
- `game.js` — all game logic (~300 lines, single file, no modules).

## Running the game

No build/install/lint/test tooling exists in this repo. To run it, open `index.html` directly or serve the directory statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There are no automated tests. Verify changes by opening the page in a browser and playing.

## Architecture (game.js)

Everything lives in module-level (`let`) state — `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc. — mutated in place by the functions below rather than passed around.

- **Board model**: a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying which piece type occupies it.
- **Pieces**: defined in `PIECES` as square matrices; the color index doubles as the lookup key into `COLORS`.
- **Rotation** (`rotateCW`): transposes + reverses rows of the shape matrix.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` against `collide()` before giving up on the rotation.
- **Collision** (`collide`): checks piece cells against board bounds and already-locked cells.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the piece into `board`, clears completed rows (shifting from the bottom up, `board.splice`/`board.unshift`), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points per row dropped, soft drop adds 1 point per row.
- **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides.

Rendering (`draw`, `drawNext`, `drawBlock`, `drawGrid`) is plain Canvas 2D — no abstraction layer. `drawBlock` takes an explicit `context` so the same function renders to both the main board canvas and the next-piece preview canvas.

Input is handled by a single `keydown` listener at the bottom of the file mapping arrow keys / `X` / `Space` / `P` to movement, rotation, drops, and pause.

## Tunable constants

Easily adjustable at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
