# Tilemap Studio

Paint a tilemap and let the tiles pick themselves — real 47-tile blob autotiling, layers, a collision layer, and export as JSON, CSV or a Tiled-compatible map. Runs entirely in your browser.

**Live:** <https://tilemap-studio.slippylabs.com/>

## What it does

- Paint, erase, flood fill, rectangle and pick, with a round brush up to 8 cells across.
- **Blob autotiling** — all eight neighbours, so inside corners work — or the 16-tile edge-only rule, or none.
- Six terrains with a solid flag, two layers, a collision overlay and a tile-index overlay.
- Maps from 4×4 to 128×128, and a demo island shaped to exercise the awkward cases.
- Export JSON (terrain grid, tile grid, rooms of the autotile table), a Tiled `.tmj`, CSV, or the raw 256-entry lookup table as JavaScript.
- The whole 47-tile sheet drawn from the masks themselves, so you can see which of the 256 neighbourhoods land on each one.

## How it works

Which tile belongs in a cell is a function of its eight neighbours and nothing else. Number those neighbours as bits and there are 256 possible neighbourhoods — but not 256 different pictures.

The rule that collapses them: a **diagonal neighbour only matters when both of the orthogonals beside it are also filled**. If grass is above me and to my right but not above-right, the corner is an outside bend and the diagonal is irrelevant. If all three are filled, the diagonal decides whether there is a notch. Clearing every diagonal whose two orthogonals are not both set takes 256 neighbourhoods down to exactly **47**.

Tile indices are never stored. They are derived from the terrain grid on demand, so a cell can never disagree with its neighbours — which is the bug autotiling exists to prevent and the bug a cached index would quietly reintroduce.

Out of bounds counts as "same terrain", so an island painted to the edge of the map does not grow a coastline along the border.

## Verification

Autotiling is unusually testable: the mapping is a pure function of eight bits, so the **entire** thing can be enumerated rather than sampled. `verify_tilemap.py` re-derives it from the rule statement, with no reference to the page's table:

- All **256** neighbourhoods normalise exactly as the rule says, collapse to exactly **47** classes, and the page's table is that set in ascending order. The normalisation is idempotent, every tile is reachable, and the counts partition 256.
- **20,168 cells** across 10 maps × 2 rules: every tile index equals the one recomputed from its neighbourhood from scratch. A control confirms the blob and edge rules disagree on 4,718 of them, so the comparison is not checking one thing twice.
- Flood fill against **`scipy.ndimage.label`**: exactly the 4-connected region of the start cell, nothing else.
- The JSON and the `.tmj` both reproduce every cell — and the `.tmj` is correctly **1-based with 0 for empty**, which if you get it wrong loads with every tile one place off in the set and looks like a corrupted tileset rather than an off-by-one.
- All 256 entries of the copied `TILE_FOR_MASK` table are correct; a rectangle is the same dragged either way; the round brush stays inside its radius.

**1,652 checks.**
