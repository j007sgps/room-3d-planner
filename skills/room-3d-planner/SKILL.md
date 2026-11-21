---
name: room-3d-planner
description: >-
  Use this skill when recreating a real-world apartment, house, or room in 3D from
  floorplan listings (e.g., Japanese madori), viewing photos, hand sketches, or tape-measure
  dimensions, cleaning measurement loop-closure errors, or building/extending an interactive
  React Three Fiber 3D interior layout planner with desktop/mobile walkthrough controls and
  preview/edit safety lock.
---

# Room 3D Planner (4-Phase Gated Workflow)

Turn raw property listings, viewing photos, and hand tape-measure dimensions into a geometrically
accurate, interactive React + Three.js (React Three Fiber / Drei) + Tailwind CSS 3D interior
space and furniture planner that works smoothly on both desktop and mobile browsers.

## Core Gate Rule

**Strictly follow the 4 phases in order and stop at each checkpoint for user confirmation.**
Never jump directly to generating final 3D code while room dimensions or ambiguous architectural
details remain unconfirmed.

---

## Phase 1: Floorplan & Photo Discovery (`格局盤點`)

1. Search the web for the property name, layout keywords (e.g., `"<物件名> 間取り"`, `"<物件名> 平面図"`, `"<property> floorplan"`), or inspect any floorplan image, sketch, and interior viewing photos provided by the user.
2. Report clearly to the user:
   - Property/layout type (e.g., `1K`, `1LDK`, studio) and approximate exclusive area (`㎡` / `帖`).
   - Relative orientation of major zones: entrance (`玄関`), wet zone (`水回り`: bath/toilet/laundry pan), kitchen (`K`), main living/sleeping room (`洋室`), closets (`CL`), bay windows (`出窓`), and balcony (`バルコニー`).
3. **Stop if exact floorplan is missing or incomplete**: Tell the user which walls or dimensions are missing and wait for them to upload sketches, viewing photos, or manual measurements.
4. **Privacy boundary**: Treat user-supplied viewing photos/videos (`img/` or similar) strictly as reference inputs. Never copy or import raw user viewing photos into `public/` or commit/deploy them unless explicitly requested. Ensure `.gitignore` excludes user photo folders.

---

## Phase 2: Measurement Cleaning & Loop Closure (`尺寸清洗與確認`)

Before writing any 3D component, read [references/measurement-cleaning.md](./references/measurement-cleaning.md) and clean the raw tape measurements:

1. **Loop Closure**: Reconcile opposing wall totals (typical `1–5 cm` tape slack/corner trim drift) by anchoring on direct wall-to-wall measurements and distributing minimal adjustments so all walls close cleanly.
2. **Orthogonalization**: Snap primary wall corners to `90°` unless an angled wall is explicitly marked.
3. **Anomaly & Ambiguity Detection**:
   - Flag any discrepancy `> 10 cm` against area/floorplan ratios — state the conflicting numbers and ask instead of guessing.
   - Verify semantic details from photos/notes: wall-mounted divider height vs. full ceiling beam, closet corner pillar (`柱`) position (NW/NE/SW/SE), deduced sill height from aligned window top frames (`sillY = topY - windowH`), and low-priority approximate zones (`水回り`).
4. **Output Confirmation Table & 2D Schematic**:
   - Present a Markdown comparison table (**原始量測值 vs. 系統修正值** with explanation).
   - Provide a labeled 2D ASCII (or SVG) floorplan with North orientation, zone labels, beams/columns, doors, and windows.
   - **Stop and wait for explicit confirmation** before starting Phase 3.

---

## Phase 3: Interactive 3D Simulator Build (`3D 建模與互動系統`)

Read [references/r3f-architecture.md](./references/r3f-architecture.md) and [references/r3f-pitfalls.md](./references/r3f-pitfalls.md) before coding. Build a Vite + React + TypeScript + R3F + Drei + Zustand + Tailwind CSS app with:

1. **Single-Truth Parametric Architecture (`cm`)**:
   - Store all room bounds, columns, beams, doors, windows, fixtures, and furniture coordinates in `cm` (converted to meters via `CM = 0.01` only in 3D renderers).
   - Model floor zones, walls, optional ceiling (`showCeiling`), cutaway front walls (`cutawayMode`), door swing arcs on the floor, clickable/toggleable closet doors (`delta` guarded against drags), utility/outlet markers, and procedural `CanvasTexture` surfaces + outdoor scenery (`day` / `sunset` / `night`).
2. **Furniture Planner & Safety Aids**:
   - Parametric multi-part 3D furniture library (including Japanese standard sizes and tension-rod storage `突っ張り収納` with vertical straddling rules above appliances like washing machines).
   - Custom furniture generator (`name`, `W × D × H`, `elevationCm`, `color`, `modelType`).
   - Drag-on-plane movement, `90°` (`R` / `Shift+R`) + fine-angle rotation, wall/grid snap, hard collision & door-swing warning highlights, 4-way real-time `cm` clearance rulers, `50`-step Undo/Redo (`Ctrl+Z` / `Ctrl+Y`), `localStorage` auto-save, and JSON export/import.
3. **Preview Mode / Edit Mode Lock (`P`)**:
   - Dedicated toggle in `TopBar` + hotkey `P`.
   - In Preview Mode: furniture clicks/drags pass straight through to `OrbitControls` (no `stopPropagation`), overlays/rulers/Inspector hide, store mutations and destructive shortcuts lock, sidebar auto-switches to view controls with lock banners, and a floating top-left badge allows 1-click return to Edit Mode.
4. **Multi-Camera + Mobile Twin-Stick Support**:
   - `3D 鳥瞰` (`OrbitControls`), `2D 俯視` (`OrthographicCamera` with pan/dolly mouse & touch mapping), and `第一人稱` (`165 cm` default eye height with `140–185 cm` slider, `115 cm` sitting / `65 cm` lying presets, realistic `60–70°` FOV, WASD + wall-sliding collision, and quick teleport spots).
   - Mobile browser UI (`100dvh`, `viewport-fit=cover`, `overscroll-behavior: none`, `touch-none`): 2-row responsive `TopBar`, auto-collapsed slide-over `LeftSidebar` drawer, bottom-sheet `InspectorPanel`, collapsible top-right eye-height pill, and dedicated **First-Person virtual analog joystick + sprint toggle** isolated by `pointerId` for simultaneous walk + 360° look.

---

## Phase 4: Build & Live Browser Verification (`啟動與雙端驗證`)

1. Run `npm run build` (`tsc -b && vite build`) and confirm exit code `0` with zero TypeScript or bundle errors.
2. Start the local Vite dev server and verify in a real headless browser across **both Desktop (`1440×900`) and Mobile (`390×844`)**:
   - **Desktop (`1440×900`)**: Zero console/page/network errors; single-row `TopBar` with no wrapped/clipped buttons; switching between `3D 鳥瞰` / `2D 俯視` / `第一人稱`; furniture selection, drag, `R` rotate, `Ctrl+Z` undo, and Preview Mode drag pass-through.
   - **Mobile (`390×844`)**: Zero horizontal page overflow (`scrollWidth <= 390`); 2-row `TopBar`; slide-over drawer open/close; bottom-sheet inspector; First-Person virtual joystick walks smoothly toward walls while right-half touch dragging rotates the camera cleanly.
3. Deliver a concise summary of verified controls, keyboard shortcuts, and touch gestures to the user.
