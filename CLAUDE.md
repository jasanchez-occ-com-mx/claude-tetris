# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris — no build, no dependencies, no `package.json`. Three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no test suite, linter, or build step.

## Architecture

All game logic lives in `game.js` (~300 lines, single global scope, `'use strict'`). Key shape:

- **Board state**: `board` is a `ROWS × COLS` matrix of integers; `0` = empty, `1–7` = piece type / color index into `COLORS`. Pieces are square matrices in `PIECES`, indexed identically — a piece's cell value doubles as its color index when merged into the board.
- **Rotation + wall kicks**: `rotateCW` transposes + reverses. `tryRotate` attempts offsets `[0, -1, 1, -2, 2]` and commits the first non-colliding one (simplified SRS — not full SRS kick tables).
- **Game loop**: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`; when it exceeds `dropInterval`, the active piece drops one row or locks. `lockPiece` → `merge` → `clearLines` → `spawn`. If a freshly spawned piece collides immediately, `endGame()` fires.
- **Ghost piece**: `ghostY()` walks `current.y` down until collision; rendered in `draw()` with `alpha=0.2` before the live piece.
- **Scoring/level**: `LINE_SCORES = [0,100,300,500,800]` × level for line clears; hard drop +2/cell, soft drop +1/row. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Pause**: `togglePause` cancels the RAF and resets `lastTime` on resume to avoid a giant `dt` jump.

## Coupling between files

`game.js` does `getElementById` on these IDs at module load — keep them in sync with `index.html` if you rename: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`.

The board canvas size in `index.html` must equal `COLS*BLOCK × ROWS*BLOCK` (currently 300×600). Changing `COLS`, `ROWS`, or `BLOCK` in `game.js` requires updating the `<canvas id="board">` `width`/`height` attributes too.

## Conventions

- Spanish for user-facing strings (overlay text, README).
- No modules, no classes — top-level `let` for mutable state, plain functions.
