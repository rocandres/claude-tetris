# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript (HTML5 Canvas + CSS). No dependencies, no build step, no package.json, no tests, no linter. README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve the directory (e.g. `python3 -m http.server`) and visit it in a browser. Verification is manual in the browser.

## Architecture

Three files, all logic in `game.js` (script-global state, `'use strict'`, no modules):

- `index.html` — DOM skeleton: `#board` canvas, `#next-canvas` preview, HUD elements, `#overlay` (pause/game over), `#theme-toggle`.
- `style.css` — theming via CSS custom properties on `:root` (dark default) overridden under `:root[data-theme="light"]`. New colors should be added as variables in both blocks.
- `game.js` — game state (`board`, `current`, `next`, `score`, ...) lives in module-level `let`s, reset in `init()`. Flow: `loop(ts)` (requestAnimationFrame, accumulates `dropAccum` against `dropInterval`) → `softDrop`/`lockPiece` → `merge` → `clearLines` → `spawn`. Rendering is a full redraw in `draw()`.

Key details:
- Pieces are defined in `PIECES` as matrices whose cell values are indices into `COLORS` (index 0 / `null` = empty). Adding a piece means adding to both arrays in the same position; `randomPiece()` picks uniformly over all `PIECES` entries. `NUT` (index 8) is the special 3×3 piece with a hollow center; `drawNutHole` renders its hole specially.
- Rotation (`tryRotate`) is clockwise with simple horizontal wall kicks `[0, -1, 1, -2, 2]`.
- Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`; scores from `LINE_SCORES` × level.
- Theme: `applyTheme` sets `document.documentElement.dataset.theme` and persists to `localStorage` (wrapped in try/catch). Canvas grid color is read from the `--grid` CSS variable at draw time, so canvas drawing must re-read CSS variables rather than hardcode theme colors. The toggle button is blurred after click so Space/Enter don't re-trigger it during play.

## CI

`.github/workflows/` contains Claude Code GitHub Actions (`claude.yml`, `claude-code-review.yml`, `claudecode-issue-formatter.yml`); branches like `claude/issue-N-...` are created by them and merged via PRs.
