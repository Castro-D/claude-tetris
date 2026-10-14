# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, build step, linter, or tests. The README and UI text are in Spanish; keep new user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

All logic lives in `game.js` (single script, `'use strict'`, global state; no modules). `index.html` supplies the DOM ids it looks up at the top of the file (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) — renaming an id requires updating both.

Key points that span the code:

- **Piece/color coupling**: `PIECES[type]` matrices store the piece's own type index (1–7) as the cell value, and `COLORS[type]` is indexed by the same value. The board stores these indices (0 = empty), so adding/reordering pieces means updating both arrays together and the `Math.random() * 7` in `randomPiece`. Index 8 (`BOMB`, a 1x1 piece) is deliberately outside that random pool.
- **Bomb piece**: `clearLines` sets `pendingBomb` each time `lines` reaches `nextBombAt` (every 5 lines); `spawn()` then creates the bomb via `createBomb()` without consuming `next`. On lock, `lockPiece` calls `explode()` (clears the clamped 3x3, collapses columns above it) before `clearLines()`, so lines formed by the collapse are cleared. Flashing/blast-zone/explosion visuals are in `drawBlock` and `draw()`.
- **Game flow**: `init()` → `spawn()` (promotes `next` to `current`, calls `endGame()` if it collides immediately) → `requestAnimationFrame(loop)`. `loop` accumulates time into `dropAccum` and gravity-drops when it exceeds `dropInterval`; `lockPiece()` = `merge()` + `clearLines()` + `spawn()`. `clearLines` also updates level and `dropInterval` (`max(100, 1000 - (level-1)*90)`).
- **Pause/game over** cancel the rAF loop (`animId`) rather than flagging inside it; `togglePause` restarts `loop` with a fresh `lastTime`. Restart reuses `init()`.
- **Rendering**: `draw()` redraws everything each frame (grid, board, ghost via `ghostY()` at alpha 0.2, current piece). The preview canvas is only redrawn in `spawn()` via `drawNext()`.
- **Sizing**: `BLOCK`, `COLS`, `ROWS` must match the `<canvas id="board">` `width`/`height` attributes in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`); they are not derived automatically.
- **Rotation**: `rotateCW` + `tryRotate` with simple horizontal kicks `[0,-1,1,-2,2]` (not SRS); only clockwise.
