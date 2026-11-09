# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no `package.json`.

## Running the game

No install/build required. Either:

```bash
start index.html       # Windows: open directly
```

or serve it locally (needed if testing anything that requires an HTTP origin):

```bash
python3 -m http.server 8000
npx serve .
```

There are no tests, linter, or build/bundle commands in this repo.

## Architecture

Three files, no modules:

- `index.html` — DOM structure: main `<canvas id="board">` (300×600, 10×20 grid of 30px blocks), a `<canvas id="next-canvas">` for the next-piece preview, HUD spans (`score`, `lines`, `level`), and a pause/game-over `overlay`.
- `style.css` — dark/retro arcade visuals.
- `game.js` — all game logic, in one file, using module-level `let` state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class or state container.

### Key mechanics in `game.js`

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1-7` identifying which piece locked there.
- **Pieces**: `PIECES` array of square matrices (one per tetromino type). Rotation is done by transposing + reversing rows (`rotateCW`), not by precomputed rotation states.
- **Collision** (`collide`): checks a shape against board bounds and existing locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns before giving up on the rotation.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Level/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece**: `ghostY()` projects where the current piece would land; drawn at `globalAlpha = 0.2`.

If changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

Game over is triggered in `spawn()` when a freshly spawned piece already collides with the board.
