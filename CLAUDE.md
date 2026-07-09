# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris. HTML5 Canvas + CSS + plain JS. No dependencies, no build step, no package.json, no tests.

## Running

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

No build/lint/test commands exist — none are configured in this repo.

## Architecture

Three files, no modules:

- `index.html` — DOM shell: `#board` canvas (300×600, 10×20 grid at `BLOCK=30`px), `#next-canvas` piece preview, HUD spans (`#score`/`#lines`/`#level`), and `#overlay` for pause/game-over.
- `style.css` — dark/retro theme, flexbox layout, no variables system.
- `game.js` — all game logic, single global scope (no classes/modules), driven by `requestAnimationFrame`.

### State

All mutable game state lives in top-level `let` bindings declared once at the top of `game.js` (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) and reset in `init()`. There is no state container/store — functions read/write these globals directly.

### Core flow

```
init() → createBoard() + randomPiece() → spawn() → requestAnimationFrame(loop)

loop(ts): accumulate dt → if dt >= dropInterval, drop piece or lockPiece() → draw() → requestAnimationFrame(loop)

keydown: move / tryRotate() / softDrop() / hardDrop() / togglePause()
```

- `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
- Pieces are square matrices (see `PIECES`); rotation is transpose+reverse via `rotateCW`.
- `tryRotate()` does wall-kick: tries offsets `[0, -1, 1, -2, 2]` before giving up.
- `collide(shape, ox, oy)` is the single collision check used everywhere (movement, rotation, spawn, ghost, gravity).
- `lockPiece()` = `merge()` current into `board` → `clearLines()` → `spawn()` next piece.
- `clearLines()` scans bottom-up, splices full rows, unshifts empty ones at top; scoring uses `LINE_SCORES = [0,100,300,500,800]` × `level`.
- Level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- Ghost piece (`ghostY()`) projects current piece straight down and is drawn at `globalAlpha = 0.2`.
- `spawn()` colliding immediately triggers `endGame()`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` × `ROWS×BLOCK`).

## Notes

- README.md (Spanish) is the canonical human-facing doc — check it for control scheme and feature descriptions before changing UX-visible behavior.
- No linter/formatter config present; match existing style (2-space indent, single quotes, semicolons).
