# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package.json. Three files: `index.html`, `style.css`, `game.js`.

## Running the game

No install/build required. Serve statically or open directly:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
# or
npx serve .
# or just open index.html directly in a browser
```

There is no test suite, linter, or build/bundle process in this repo.

## Architecture

All game logic lives in `game.js` (single file, no modules). Key pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix (20×10); each cell is `0` (empty) or a color index 1–7 identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices, indexed by color/type 1–7. Rotation is done via `rotateCW` (transpose + reverse rows), not by storing pre-rotated states.
- **Collision** (`collide`): checks board bounds and overlap with already-locked cells for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded, otherwise calls `lockPiece()`.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top; updates score/lines/level and recalculates `dropInterval`.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. Level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn with `globalAlpha = 0.2`.
- **State machine**: module-level mutable state (`board, current, next, score, lines, level, paused, gameOver, ...`) reset by `init()`. `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires.

Rendering: `draw()` redraws the whole board canvas every frame (grid → locked blocks → ghost → current piece). `drawNext()` renders the preview piece on a separate small canvas (`next-canvas`).

Input: a single `keydown` listener switches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause), guarded by `paused`/`gameOver` checks.

## Tuning constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

## CI

`.github/workflows/claude.yml` and `.github/workflows/claude-code-review.yml` run Claude Code against pushes/PRs (installed via `/install-github-app`). `.github/workflows/claude-issue-triage.yml` runs on issue `opened`/`edited`: it labels the issue from the repo's existing labels only and posts a short Spanish-language markdown triage comment (type, severity, area, summary) — it does not read the source code or propose an implementation.
