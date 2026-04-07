# Lawn God — Game Specification v0.1

> A retro puzzle game about optimal lawn mowing routes, inspired by Sokoban, Paperboy, and the Königsberg bridge problem.

---

## Concept

You are a teenage entrepreneur starting a lawn-mowing business with a basic push mower. Each level is a grid of grass tiles that must be fully mowed. The fewer moves you use, the higher your score and the bigger the tip. Overlapping already-cut grass wastes fuel. The underlying puzzle is a **Hamiltonian path / Eulerian circuit problem** on a grid graph — and the starting tile is the critical first decision.

---

## Repository Structure

```
lawn-god/
├── index.html          # Single-file game (current build)
├── SPEC.md             # This document
├── README.md           # Player-facing intro + how to play
├── LICENSE             # MIT
└── levels/
    └── levels.js       # Level data (extracted from index.html for future builds)
```

For GitHub Pages: enable Pages on the `main` branch root. The game runs as a pure static site — no build step, no dependencies beyond two Google Fonts.

---

## Core Mechanics

### Grid

- Rectangular grid of square tiles, typically 3–8 tiles wide/tall.
- Each tile is one of: **grass** (mowable), **water** (impassable), **flower bed** (impassable), or future obstacle types.
- The player navigates one tile per move, orthogonally (no diagonals).

### Starting Position

- **The player clicks (or taps) any grass tile to place the mower.** This is a deliberate first decision — the optimal starting tile is often non-obvious.
- During placement mode, hovering shows a ghost mower preview on valid tiles.
- Once placed, movement is via arrow keys / D-pad only. The start tile counts as move 1.

### Movement

- Arrow keys (↑ ↓ ← →) or on-screen D-pad.
- Each key press moves the mower one tile in that direction.
- Moving onto an **uncut** tile: cuts it, increments move counter.
- Moving onto an **already-cut** tile: incurs a **fuel penalty** ($0.50), move still counts.
- Moving into a wall, obstacle, or out of bounds: blocked, no move consumed.
- `Z` key: undo last move (unlimited undos, no penalty).
- `R` key: reset level.

### Win Condition

All mowable tiles cut. Score popup appears immediately.

---

## Scoring

Each level has the following parameters:

| Parameter | Description |
|-----------|-------------|
| `fee`     | Base contract payment, always awarded on completion |
| `par`     | Minimum theoretically possible moves (precomputed) |
| `tip`     | Maximum tip, awarded only on perfect (≤ par) completion |

**Payout formula:**

```
earned = fee + parBonus + tip - fuelPenalty - overlapFine
```

| Component | Condition | Amount |
|-----------|-----------|--------|
| Contract fee | Always | `fee` |
| Par bonus | moves ≤ par | `fee × 0.5` |
| Par bonus | moves ≤ par+2 | `fee × 0.25` |
| Par bonus | moves > par+2 | `$0` |
| Tip | moves ≤ par | `tip` (full) |
| Tip | moves ≤ par+2 | `tip × 0.6` |
| Tip | moves ≤ par+5 | `tip × 0.3` |
| Tip | moves > par+5 | `$0` |
| Fuel penalty | moves > par | `(moves − par) × $0.40` |
| Overlap fine | per re-cut tile | `$0.50` |

Grade tiers: **PERFECT** (≤ par) · **GREAT** (par+1–2) · **GOOD** (par+3–5) · **DONE** (par+6+)

Running balance accumulates across levels in a single session (no persistence yet).

---

## Level Design

### The 20 Starter Levels

All levels have hand-verified par values. Par is the length of a known optimal Hamiltonian path (or near-optimal if Hamiltonian path doesn't exist, requiring minimal backtrack).

| # | Name | Grid | Par | Notes |
|---|------|------|-----|-------|
| 1 | Starter yard | 4×3 | 11 | Pure rectangle |
| 2 | Wide strip | 6×2 | 11 | Boustrophedon optimal |
| 3 | L-shaped lot | 5×4 (−2) | 15 | Corner removal |
| 4 | Square plot | 5×5 | 24 | First real challenge |
| 5 | Corner beds | 5×4 (−4 beds) | 14 | Ornamental obstacles |
| 6 | Pond job | 5×5 (−1 water) | 23 | Central obstacle |
| 7 | Twin ponds | 6×4 (−2 water) | 19 | Two obstacles |
| 8 | U-shape | 5×4 (−5) | 10 | Classic Euler shape |
| 9 | Central island | 6×5 (−6) | 22 | Mixed obstacle types |
| 10 | Zigzag strip | 7×3 (−5) | 14 | Staggered blocks |
| 11 | Estate garden | 6×6 (−4 water) | 31 | Large open grid |
| 12 | Diagonal path | 6×5 (−5) | 22 | Diagonal obstacle line |
| 13 | Four corners | 6×6 (−6) | 29 | Symmetric layout |
| 14 | Corridor | 7×5 (−6) | 23 | Two internal walls |
| 15 | Checkerboard | 5×5 (−12 beds) | 12 | Sparse grid |
| 16 | S-curve | 7×4 (−7) | 19 | Shape hints solution |
| 17 | Ring road | 6×6 (−12) | 19 | Perimeter-only mowing |
| 18 | Notched square | 6×6 (−8) | 27 | 4-sided notches |
| 19 | The maze | 7×5 (−9) | 26 | Grid of posts |
| 20 | Grand estate | 8×6 (−8) | 39 | Final challenge |

### Level Data Format

```js
{
  name: 'Pond job',
  w: 5, h: 5,
  obs: [ { x: 2, y: 2, t: 'water' } ],  // t: 'water' | 'bed' | omit for generic block
  par: 23,
  fee: 16,   // base contract payment in $
  tip: 12,   // max tip in $
  desc: 'Watch out for the pond!'
}
```

---

## The Graph Theory Behind Par

Each mowable tile is a **node**. Two nodes are connected by an **edge** if they are orthogonally adjacent. The optimal mowing route is a **Hamiltonian path** — visiting every node exactly once.

Key insights surfaced in hints:
- A node's **degree** = number of mowable neighbours.
- Nodes with **odd degree** are forced endpoints. If there are exactly 2 odd-degree nodes, the optimal path must start at one and end at the other.
- If there are **0 odd-degree nodes**, an Euler circuit exists and you can start anywhere.
- More than 2 odd-degree nodes means some backtracking is unavoidable — par accounts for this.

For levels 1–20, par was established by hand-solving. Future levels (21+) will use an approximate solver.

---

## Planned Features (Roadmap)

### Phase 2 — Hazards & Life

- **Hedgehog** — wandering NPC that moves one tile per player move (random walk avoiding obstacles). Colliding costs $2 vet fee. Crossing its path is safe; running into it is not.
- **Customer watching** — visible countdown/patience meter; taking too long reduces tip incrementally.

### Phase 3 — Upgrades

Purchasable between levels from your balance:

| Upgrade | Cost | Effect |
|---------|------|--------|
| Wide mower | $15 | Cuts 2-tile swath (current tile + one to the side, configurable) |
| Ride-on mower | $30 | Faster feel, but costs $0.10 extra fuel per move |
| Robot mower | $50 | Auto-solves the current level (demo only, no earnings) |
| Fuel can | $5 | Removes penalty for next 3 overlap moves |

### Phase 4 — Level Generation

- Approximate solver (greedy Hamiltonian path with backtrack limit) to compute par for procedurally generated levels.
- Level editor: place obstacles on a grid, let the solver compute par, save as JSON.
- Seed-based random level generation for daily challenges.

### Phase 5 — Polish

- Sound effects (mower hum, grass cut, coin pickup) — Web Audio API, no assets needed.
- High score persistence via `localStorage`.
- Shareable completion cards (canvas snapshot + score).
- Mobile swipe gestures for movement.

---

## Technical Notes

- **Pure HTML5** — single `index.html`, no build toolchain.
- **Canvas 2D API** for all rendering; pixel art aesthetic via `image-rendering: pixelated`.
- **Fonts**: Press Start 2P (headings/UI) + VT323 (HUD numbers) via Google Fonts — add a local fallback for offline play.
- **No frameworks, no dependencies** beyond fonts.
- State is held in a plain JS object (`gs`); undo uses JSON snapshot stack.
- Game loop via `requestAnimationFrame` (used only for overlap flash animation; render is otherwise event-driven).

### GitHub Pages Deployment

```bash
git init
git add .
git commit -m "Initial Lawn God build"
git remote add origin https://github.com/YOUR_USERNAME/lawn-god.git
git push -u origin main
# Then: GitHub repo Settings → Pages → Branch: main / root → Save
# Live at: https://YOUR_USERNAME.github.io/lawn-god/
```

---

## Known Issues / Next Steps

- Par values for levels 8, 13, 17–19 should be re-verified by a solver (currently hand-estimated).
- Wide mower upgrade needs a direction-tracking system (last move direction determines swath side).
- No `localStorage` persistence — balance resets on page reload.
- Canvas scaling on very small screens (< 400px wide) clips the largest levels (8-wide grids).

---

*Lawn God v0.1 — built with Claude, April 2026*
