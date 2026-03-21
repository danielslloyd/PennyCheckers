# Penrose Tile Game — Requirements Spec

## Overview

A two-page browser-based game built on a Penrose P2 tiling. Page 1 is a visual demo of the tiling and its graph. Page 2 is a two-player tactical game played on that graph, with an optional AI opponent.

---

## Page 1 — Penrose Tiling Demo (`index.html`)

### Tiling Generation

- Generate a Penrose P2 (kite and dart) tiling via recursive deflation, starting from a suitable initial configuration (e.g., a sun or ring of kites).
- Perform enough deflation iterations to produce ~100–300 visible tiles filling the canvas.
- Render tiles as SVG or Canvas 2D. Each tile should be clearly outlined; kites and darts must be visually distinguishable by fill color.
- Center and scale the tiling to fit the viewport.
- Tiling generation must be **seeded**: accept a seed string that deterministically produces the same tiling layout every time. Display the current seed and provide a "New Random Seed" button that regenerates with a new random seed and updates the URL hash (e.g. `#seed=abc123`) so the board is shareable.

### Graph Construction

- After tiling generation, build an adjacency graph where:
  - **Nodes** = individual tiles
  - **Edges** = tiles that share a full edge (not merely a vertex)
- Edge-sharing detection: two tiles share an edge if they have exactly two vertices in common (within a suitable floating-point epsilon).
- Store the graph as an adjacency list: `Map<tileId, Set<tileId>>`.

### Demo Visualization

- Clicking a tile highlights it and draws visible lines to all its edge-adjacent neighbors.
- Display a small info panel showing: tile type (kite/dart), tile ID, and neighbor count.

---

## Page 2 — Two-Player Game (`game.html`)

### Board Setup

- Reuse the same Penrose tiling and graph as Page 1, driven by the same seed system. The seed is shown and shareable via URL hash.
- Place **Team A** (gold) and **Team B** (teal) pieces on the board at game start, spread across opposing regions of the tiling.
- Each team starts with **4 pieces**, composed of a fixed mix of piece types (see Piece Types below).

### Piece Types

Each team has the same composition of three piece types, distinguished visually by icon or shape:

| Type    | Max Steps per Turn | Count per Team |
|---------|--------------------|----------------|
| Scout   | 3                  | 2              |
| Soldier | 2                  | 1              |
| Guard   | 1                  | 1              |

- A piece's max steps is its permanent attribute — it never changes.
- On each turn, the player may move a piece **up to** its max steps (i.e. a Scout may move 1, 2, or 3; a Guard must move exactly 1).

### Turn Structure

- **Strict alternating turns**: Team A moves one piece, then Team B, and so on.
- Team A goes first.
- On each turn, the active player selects one of their pieces, then selects a destination tile within range.
- A piece must move each turn (no passing). A piece must move at least 1 step.

### Movement Rules

- Movement follows graph adjacency: each step moves to an edge-adjacent tile.
- Pieces may pass through occupied tiles (friendly or enemy) freely during a multi-step move.
- The destination tile may be occupied.
- Reachable tiles for a selected piece (all tiles 1–N steps away, where N = piece's max steps) must be highlighted on selection.

### Capture Rules

- **After each individual move**, check every piece on the board for capture.
- A piece is captured if **strict majority** of its graph neighbors are occupied by enemy pieces.
  - Strict majority = more than half of the tile's total neighbor count.
  - Example: a tile with 4 neighbors is captured if 3 or more are enemy-occupied.
  - Empty neighbors and friendly-occupied neighbors count against the majority threshold.
- Captured pieces are immediately removed from the board.
- Multiple pieces may be captured in the same post-move check.
- The piece that just moved can itself be captured if the above condition is met.

### Win Condition

- The game ends when one team has zero pieces remaining.
- Display a win banner with the winning team name and a "Play Again" button that resets to starting positions (same seed, same layout).

### AI Opponent (Single-Player Mode)

- A toggle at game start lets the player choose: **Two Player** (local hot-seat) or **vs AI** (player is Team A, AI is Team B).
- The AI is a greedy bot that evaluates all legal moves and picks the one that maximizes the following priority, in order:
  1. Moves that immediately capture one or more enemy pieces.
  2. Moves that minimize the number of its own pieces that are capture-vulnerable after the move.
  3. Moves that maximize the number of enemy pieces that will be capture-vulnerable after the move.
  4. Random tiebreak among equally scored moves.
- The AI takes its turn automatically after a short delay (~600ms) so the player can see what happened.

---

## Tile Ownership Coloring

- Each tile tracks the last team to occupy it (or neutral if never occupied).
- Tiles gradually tint toward the last occupier's color (gold or teal) at low opacity, giving a visual territory feel.
- Tint resets to neutral if the tile has been unoccupied for more than 5 turns.

---

## Move History Log

- A scrollable sidebar (or panel) displays a log of every move made in the current game.
- Each entry shows: turn number, team, piece type, and destination tile ID.
- Example: `T12 · Gold Scout → #247`
- Captures are annotated inline: `T12 · Gold Scout → #247 (captured Teal Guard)`
- The log auto-scrolls to the latest entry.

---

## UI / UX

- **Tile highlighting states:**
  - Selected piece: distinct highlight color
  - Reachable destination tiles: a second highlight color
  - Tiles that would result in immediate self-capture: optionally flagged with a warning tint (nice-to-have)
- **Status bar**: whose turn it is, piece counts per team, and current seed.
- **Capture animation**: briefly flash or fade out captured pieces before removal.
- Navigation link between `index.html` and `game.html` on both pages.

---

## Technical Constraints

- **Single-file HTML** per page (inline CSS and JS), or two HTML files in the same directory.
- No build step, no npm — plain HTML/CSS/JS only.
- No external dependencies except optionally one CDN-hosted library (e.g. D3 for SVG) if needed.
- Must run correctly from `file://` or a simple static server.
- Target: modern desktop Chrome/Firefox. Mobile not required.
- Seed state stored in and read from `window.location.hash` so links are shareable.

---

## File Structure

```
index.html    # Page 1: Penrose tiling demo
game.html     # Page 2: Two-player / vs-AI game
```
