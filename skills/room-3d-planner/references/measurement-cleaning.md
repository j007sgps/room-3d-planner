# Phase 2 Reference: Measurement Cleaning, Loop Closure & Floorplan Disambiguation

Hand tape measurements in furnished or empty apartments inevitably accumulate small cumulative errors (corner trim, baseboards, door frames, tape sag). Clean and freeze geometry **before** writing any Three.js code.

---

## 1. Coordinate System Convention

Use a consistent right-handed Three.js coordinate convention backed by **centimeters (`cm`)** everywhere in data structures:

- **`+X` = East (Right in 2D Top-Down)**
- **`-X` = West (Left in 2D Top-Down)**
- **`+Z` = South (Down in 2D Top-Down)**
- **`-Z` = North (Up in 2D Top-Down)**
- **`+Y` = Up (Floor = `0`, Ceiling = e.g. `250 cm`)**

Convert to Three.js scene meters only at render time via a single constant (`export const CM = 0.01`). Putting `floorplan` coordinates directly in `(x1..x2, z1..z2)` `cm` makes wall closure math trivial and ensures `2D 俯視` aligns intuitively with North-up floorplans.

---

## 2. Geometry Cleaning Rules

### A. Orthogonal Alignment
- Assume primary walls meet at `90°` unless the user explicitly marks diagonal/angled walls.
- Express every room zone as clean axis-aligned `(xMin, xMax, zMin, zMax)` rectangles sharing common partition wall thicknesses (typical partition `8–12 cm`, structural wall `15–20 cm`).

### B. Loop Closure (1–5 cm Drift)
When opposing walls or cumulative corridor runs disagree by `1–5 cm`:
1. **Anchor on high-confidence direct spans**: Unobstructed wall-to-wall measurements inside the main living room take precedence over multi-segment hallway sums (which include door jambs, entry step offsets, or recessed molding).
2. **Distribute minimal residual error**: Absorb remaining `1–3 cm` slack in secondary transition zones (e.g., closet door trim thickness, entrance shoe-zone `土間` depth, or partition thickness) rather than distorting rectangular rooms into trapezoids.
3. **Verify zero gaps / double-counted partitions**: Check `left_zone_width + partition + right_zone_width === outer_span`.

### C. Anomaly Detection (`> 10 cm` Mismatch)
If any measurement conflicts with total area (`㎡` / `帖`), opposing wall sums, or photo proportions by **more than `10 cm`**:
- **Do NOT guess or silently alter the number.**
- Flag the conflict explicitly with two possible interpretations (e.g., measured from inside door jamb vs. wall corner, or `cm` digit typo) and ask the user to confirm.

---

## 3. Architectural Disambiguation Checklist (Inspect Photos Carefully)

Common misunderstandings from rough notes and property floorplans that cause modeling errors:

1. **Corner Columns (`柱`) Inside Closets / Corners**:
   - Floorplans often label an interior structural column (`30×30 cm` or `40×40 cm`) inside a closet or room corner without stating *which* corner (NW vs. NE vs. SW vs. SE).
   - Cross-check photos for where the clothes rod terminates and which side wall steps inward (`有效收納寬 = 總寬 - 柱寬`).
2. **Low Divider Panels vs. Full Overhead Beams (`垂れ壁 / 梁`)**:
   - A height note near a kitchen/fridge area (e.g., `H = 185 cm`) might refer to a vertical plastic/wood partition screen rather than a ceiling bulkhead dropping down over the walkway. Check viewing photos before adding overhead `beam` colliders!
3. **Unmeasured Window Sill Height (`腰窓`)**:
   - If the top frame of a side bay window (`出窓`) aligns horizontally with the top of the patio sliding door (`掃き出し窓`), deduce sill elevation by: `sillY = alignedTopY - windowHeight`.
4. **Door Frame vs. Clear Opening vs. Swing Arc**:
   - Record clear passage width, frame thickness (`2–4 cm`), door height (typically `200 cm`), and hinge side + swing direction (into hall vs. into room) so floor swing arcs warn against blocking doors with furniture.
5. **Low-Priority Wet Zones (`水回り`)**:
   - When the user focuses on living-room furniture layout and says bath/toilet measurements are approximate, fit standard modular unit proportions (`1116` / `1216` unit bath, `63×63 cm` washing machine waterproof pan) cleanly into the remaining structural footprint.

---

## 4. Required Phase 2 Output Template

Always stop and present these three items before starting Phase 3:

### 1) Original vs. Corrected Comparison Table
| Zone / Segment | 原始量測值 (cm) | 系統修正值 (cm) | 差異 / 修正說明 |
| :--- | :--- | :--- | :--- |
| Main Room (Living) | e.g., 259 × 386 | 259 × 386 | High-confidence anchor span (unchanged) |
| Ceiling Height | 250 | 250 | Verified uniform height |
| Bay Window Sill | Unmeasured | 84 (top 184) | Aligned top `184 cm` minus `100 cm` frame height |

### 2) Labeled 2D ASCII (or SVG) Schematic
Draw a proportional ASCII diagram showing North orientation (`↑ N`), room widths/depths, doors (`D`), closets (`CL`), pillars (`█`), windows (`═`), and balcony (`BALCONY`).

### 3) Confirmation Prompt
Summarize any deduced dimensions (closet height, sill height, column corner, partition height) in bullet points and wait for explicit approval before Phase 3.
