# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

**Running the game:**
- Local: `python3 -m http.server 8000` or `npx serve .`, then visit `http://localhost:8000`
- Direct: Open `index.html` in a browser (also works without a server)

**No build process, no dependencies.** Pure HTML5, CSS3, and vanilla JavaScript.

## Architecture Overview

The Tetris implementation uses a simple three-file architecture:

1. **`index.html`** — DOM structure with two canvas elements (main board at 300×600px, next-piece preview at 120×120px) and a sidebar with score/lines/level display and an overlay for pause/game-over states.

2. **`style.css`** — Dark arcade aesthetic with flexbox layout. Overlays use `backdrop-filter: blur` for the pause/game-over screens. All styling is self-contained; no external fonts or libraries.

3. **`game.js`** — Complete game logic (~305 lines):
   - **Game state**: `board` (2D array where cells store 0 or color indices 1–7), `current`/`next` pieces, score/lines/level, pause/game-over flags.
   - **Board model**: `ROWS=20 × COLS=10` grid. `board[y][x]` is 0 (empty) or a piece type ID (1–7).
   - **Piece system**: Seven standard Tetris pieces defined in `PIECES[]` as 3–4 cell matrices. Each spawns centered horizontally.
   - **Game loop**: `requestAnimationFrame`-based with accumulator pattern (`dropAccum`) to handle piece gravity independent of frame rate. Drop interval starts at 1000ms and decreases with level.

## Key Code Sections

| Function | Purpose | Location |
|----------|---------|----------|
| `createBoard()` | Initialize empty 20×10 grid | game.js:45 |
| `collide(shape, ox, oy)` | Check piece/wall/floor collisions | game.js:55 |
| `rotateCW(shape)` | 90° clockwise rotation via transpose | game.js:68 |
| `tryRotate()` | Attempt rotation with wall kicks (±1, ±2 columns) | game.js:77 |
| `merge()` | Lock piece to board after landing | game.js:89 |
| `clearLines()` | Detect/remove full rows, update level/speed | game.js:96 |
| `ghostY()` | Calculate landing position for ghost piece preview | game.js:115 |
| `hardDrop()` / `softDrop()` | Instant/accelerated drop with scoring | game.js:121–136 |
| `loop(ts)` | Main game loop; accumulates time and triggers piece drops | game.js:243 |
| `draw()` | Render board, grid, ghost, and active piece | game.js:188 |
| `drawNext()` | Render next-piece preview on separate canvas | game.js:210 |

## Game Mechanics

- **Gravity**: Piece falls every `dropInterval` ms (starts 1000ms, decreases ~90ms per level).
- **Wall kicks**: When rotation fails due to collision, tries offsets `[0, -1, 1, -2, 2]` before rejecting.
- **Scoring**: Single/double/triple/tetris = 100/300/500/800 points (×level). Soft drop = 1 pt/row, hard drop = 2 pts/cell.
- **Line clearing**: Recalculates from bottom up; splices completed rows and inserts blank row at top. Updates level every 10 lines.
- **Ghost piece**: Semi-transparent preview (alpha=0.2) showing final position; recalculated each frame.

## Customization Points

The following constants in `game.js` are tunable:

| Constant | Default | Notes |
|----------|---------|-------|
| `COLS` | 10 | Board width. Update canvas width to `COLS × BLOCK`. |
| `ROWS` | 20 | Board height. Update canvas height to `ROWS × BLOCK`. |
| `BLOCK` | 30 | Pixel size per cell. Sync canvas dimensions when changed. |
| `COLORS` | 7 colors | Color palette for pieces (indices 1–7). Must match `PIECES` length. |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points for 1–4 line clears (array[0] unused). |
| Initial `dropInterval` | 1000 ms | Starting fall speed. |

> When changing `COLS`/`ROWS`/`BLOCK`, keep canvas `width` and `height` in sync: `width = COLS × BLOCK`, `height = ROWS × BLOCK`.

## Event Handling

Keyboard input via `keydown` listener (game.js:277):
- Arrow keys: move/soft-drop
- Arrow Up / X: rotate
- Space: hard drop
- P: pause/resume
- Pause key is always active; other inputs are blocked during pause/game-over.
- Restart button calls `init()` to reset all state.

## Testing / Iteration

Since there's no build step:
1. Edit any file.
2. Refresh the browser tab (`F5` or `Cmd+R`).
3. Changes are instant.

For tweaking constants, modify `game.js` and reload. For layout/styling, edit `style.css` and reload.

## Notes for Future Work

- **Piece rotation**: Uses simple transpose + row-reverse; considers the piece a 4×4 bounding box (padded with zeros for smaller shapes).
- **Collision detection**: Iterates piece cells and checks board bounds and occupancy; does not account for negative Y (pieces above the visible board).
- **Canvas rendering**: Grid is drawn every frame with `globalAlpha` reset between draws; colors are hex strings looked up from `COLORS[]`.
- **Ghost piece**: Not stored in `board`; recomputed per frame by simulating downward movement until collision.
