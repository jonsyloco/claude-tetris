# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server, e.g.:

```bash
python3 -m http.server 8000
npx serve .
```

There is no build, lint, or test tooling in this repo — changes to `game.js` take effect on browser reload.

## Architecture

Three files, no modules/bundler — `index.html` loads `game.js` as a single classic script that runs immediately (`init()` is called at the bottom of the file, no `DOMContentLoaded` wrapper).

- **`index.html`** — DOM structure: the main `<canvas id="board">` (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), a side panel with score/lines/level and a `<canvas id="next-canvas">` preview, and a shared overlay div reused for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade theme.
- **`game.js`** — all game logic, structured around:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying the locking piece's type.
  - **Pieces**: defined as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table — it works generically for any square matrix shape.
  - **Collision** (`collide`): bounds + board-overlap check, used for movement, rotation, and ghost-piece projection.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide, else the rotation is discarded.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row when it exceeds `dropInterval`.
  - **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
  - **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece** (`ghostY`): projects the current piece straight down until collision, drawn at `globalAlpha = 0.2`.

All mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) lives in module-level `let` bindings reassigned by `init()` — there is no state container object.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`. If `COLS`, `ROWS`, or `BLOCK` change, the `<canvas id="board">` `width`/`height` attributes in `index.html` must be updated to match (`COLS × BLOCK` and `ROWS × BLOCK`).

## Controls

| Key        | Action                  |
| ---------- | ----------------------- |
| `←` / `→`  | Move piece horizontally |
| `↑` or `X` | Rotate clockwise        |
| `↓`        | Soft drop               |
| `Space`    | Hard drop               |
| `P`        | Pause/resume            |
