# Phase 3 Reference: React Three Fiber Architecture & UX Patterns

Proven architecture for building a responsive desktop + mobile 3D interior room simulator with **React 18 + Vite + TypeScript + Three.js (`@react-three/fiber`, `@react-three/drei`) + Zustand + Tailwind CSS + Lucide icons**.

---

## 1. Modular File Structure

```text
src/
├── data/
│   ├── floorplan.ts         # Single source of truth: DIMS, COLUMNS, BEAMS, TELEPORT_SPOTS (all in cm)
│   └── furnitureCatalog.ts  # Types, Japanese standard catalog + tension-rod items, starter DEMO_LAYOUT
├── utils/
│   ├── geometry.ts          # Collision (walls/columns/beams/items), door swing check, snapping, 4-way rulers, FP move collision
│   ├── textures.ts          # Procedural HTML5 Canvas textures (oak wood, wallpaper, tile, facade, sky domes)
│   └── fpInput.ts           # Zero-re-render mutable bridge for mobile virtual joystick & sprint
├── store/
│   └── useRoomStore.ts      # Zustand store: camera/UI toggles, interactionMode, 50-step undo/redo, localStorage persistence
├── components/
│   ├── RoomModel.tsx        # Floors, walls, cutaway hiding, ceiling, windows, doors, arcs, clickable closet doors, markers
│   ├── OutdoorScenery.tsx   # Sky dome + street/roof context + animated distant details (optional toggle)
│   ├── FurnitureMesh.tsx    # Multi-part procedural 3D furniture models + selection wireframes/rings/badges
│   ├── FurnitureLayer.tsx   # Pointer drag plane, cursor, collision map, 4-way DistanceRulers
│   ├── CameraController.tsx # FirstPersonRig, Top2DRig (Ortho), Orbit3DRig (Persp)
│   ├── SceneCanvas.tsx      # <Canvas>, time-of-day lighting presets, onPointerMissed deselect guard
│   └── ui/
│       ├── TopBar.tsx       # Responsive header (2 rows < md, 1 row md+): camera, Edit/Preview toggle, view toggles, JSON I/O
│       ├── LeftSidebar.tsx  # Catalog, custom generator, view/eye-height/snap/scenery controls (drawer on mobile)
│       ├── InspectorPanel.tsx # Selection editor + clearance readout (right float on md+, bottom sheet on < md)
│       ├── MobileFpControls.tsx # Virtual analog joystick + sprint toggle for mobile/touch First-Person mode
│       └── ShortcutsModal.tsx   # Keyboard + mobile touch gesture guide
└── App.tsx
```

---

## 2. Geometry, Collision & Real-Time Clearance (`geometry.ts`)

1. **Rotated AABB Footprint**: Always compute world footprint extents `halfW`, `halfD` from `(|w*cosθ| + |d*sinθ|) / 2` and `(|w*sinθ| + |d*cosθ|) / 2` so `45°` or `90°` rotations behave accurately.
2. **Vertical Range Overlap + Smart Straddling**:
   - Check both XZ and Y overlap (`elevationCm .. elevationCm + heightCm`) against overhead beams (`BEAMS`) and stacked items.
   - Support open-bottom furniture such as **`突っ張り洗衣機上架` (`tensionLaundry`)** straddling a floor appliance (`washer` / box with `maxY <= 115 cm`) without triggering false hard collisions whenever the rack height is tall enough (`>= 2.1 m`).
3. **Two-Tier Collision Feedback**:
   - **Hard collision** (`#ef4444` red wireframe + alert pill): overlapping structural walls/columns/beams or intersecting another furniture volume.
   - **Soft warning** (`#f59e0b` amber wireframe): intruding into a door swing arc (`showDoorArcs`).
4. **4-Way Cardinal Rulers (`computeClearances`)**:
   - Cast N/S/E/W rays from the selected item's edges to the nearest opposing structural obstacle or neighbor item (`distanceCm`) to display real-time walkway clearance (`< 50 cm` highlighted in amber as tight walkway).

---

## 3. State Management, Undo/Redo & Preview/Edit Mode (`useRoomStore.ts`)

1. **50-Step Undo / Redo (`past` / `future`)**:
   - Snapshot only once when starting a continuous gesture (`initialItemsSnapshot` on drag start, `onPointerDown={() => pushHistorySnapshot()}` on rotation sliders), passing `{ recordHistory: false }` during continuous move/drag frames.
   - Discrete actions (`add`, `remove`, `duplicate`, `90° rotate`, `clear`, `loadDemo`, `import`) push history cleanly and persist immediately to `localStorage`.
2. **Preview Mode vs. Edit Mode (`interactionMode: 'edit' | 'preview'`, hotkey `P`)**:
   - **Defensive store guards**: Return early in all item mutation actions (`add*`, `updateItem`, `removeItem`, `rotateSelected`, `clearItems`, `importItems`, `undo`, `redo`) and ignore `selectItem(id)` when `id !== null` in Preview Mode.
   - **Pointer pass-through (`FurnitureLayer.tsx`)**: When `!canInteract` (`interactionMode === 'preview' || cameraMode === 'firstPerson'`), pointer handlers return immediately **without** calling `e.stopPropagation()`. Left-click drags directly on top of furniture pass cleanly through to `OrbitControls`!
   - **Visual hygiene**: Suppress wireframes, footprint rings, warning badges, `DistanceRulers`, and `InspectorPanel` in Preview Mode; auto-switch `LeftSidebar` to the view settings tab (`'view'`) and disable catalog/custom controls behind a 1-click "切換回編輯模式" banner; show a compact top-left floating lock pill.

---

## 4. Camera Rigs & Mobile Twin-Stick First-Person

1. **Desktop + Mobile Camera Rigs (`CameraController.tsx`)**:
   - **`3D 鳥瞰` (`Orbit3DRig`)**: `PerspectiveCamera` (`fov=46`) + damped `OrbitControls`, `maxPolarAngle = PI/2 - 0.03`. Enable `cutawayMode` by default so front walls hide automatically from outside.
   - **`2D 俯視` (`Top2DRig`)**: `OrthographicCamera` with responsive zoom, `enableRotate={false}`, explicit `mouseButtons={{ LEFT: PAN, MIDDLE: DOLLY, RIGHT: PAN }}`, and `touches={{ ONE: TOUCH.PAN, TWO: TOUCH.DOLLY_PAN }}`.
   - **`第一人稱` (`FirstPersonRig`)**: Auto-enables ceiling; `fov=65` (`50–80°` slider) to preserve realistic room enclosure; eye height `165 cm` default (`140–185 cm` slider + `115 cm` sitting / `65 cm` lying presets); `resolveFirstPersonMovement` splits X and Z checks so walking diagonally slides smoothly along walls without clipping out.
2. **Mobile Virtual Analog Joystick + Multi-Touch Look (`fpInput.ts` + `MobileFpControls.tsx`)**:
   - Store `{ forward, strafe, sprint }` in a plain module-level mutable object (`fpVirtualMove`) polled at 60 FPS inside `FirstPersonRig`'s `useFrame` — never trigger Zustand/React re-renders at 60 Hz during thumb drags!
   - Preserve analog magnitude (`len > 0.05`; normalize only when `len > 1`) so slight thumb deflections walk slowly and full deflections reach walking/sprint speed (`155` / `240 cm/s`).
   - Track `pointerId` strictly in both components: `MobileFpControls` captures its joystick pointer (`pointerIdRef`) while `FirstPersonRig` tracks only `activeLookPointerId` on the WebGL canvas, letting users walk with their left thumb while turning `360°` with their right thumb simultaneously.
   - Always reset `fpVirtualMove` on `pointerup`, `pointercancel`, `lostpointercapture`, and component unmount.
   - Auto-display `MobileFpControls` using `window.matchMedia('(max-width: 1023px), (pointer: coarse)')`.

---

## 5. Mobile-First Responsive Layout Rules

- **Viewport (`index.html` & `index.css`)**: Use `viewport-fit=cover`, `user-scalable=no`, `height: 100dvh`, `overscroll-behavior: none`, `.no-scrollbar`, and `touch-none` on the 3D wrapper so mobile browsers never bounce, pull-to-refresh, or get clipped by browser bars.
- **2-Row Mobile TopBar (`TopBar.tsx`)**: Wrap Row 1 (room-number badge, `3D/2D/漫遊`, `編輯/預覽`) in `flex ... md:contents` so mobile shows a clean 2-row header (Row 2 is a `.no-scrollbar` horizontal strip) while `md+` keeps a single-row flex bar with zero layout duplication.
- **Auto-Collapsed Slide-Over Drawer (`LeftSidebar.tsx`)**: Initialize `collapsed` via `window.innerWidth < 768`; on `< md`, render as a `fixed inset-y-0 left-0 z-40` drawer with backdrop and auto-close after adding furniture or triggering teleport spots. When collapsed in Preview Mode, offset down (`top-14`) below the Preview lock pill (`top-3`).
- **Bottom-Sheet Inspector & Collapsible Eye-Height Pill**: Dock `InspectorPanel` as `inset-x-2 bottom-2 max-h-[45dvh]` on `< md` (`top-3 right-3 w-72` on `md+`), hide desktop bottom keyboard hint pill (`hidden md:block`), and default the First-Person overlay to a tiny `視高 165cm ▾` pill on `< md`.

---

## 6. Zero-CDN Procedural Materials (`textures.ts`)

Generate cached `THREE.CanvasTexture` assets in memory for oak wood planks (`RepeatWrapping`), subtle painted wall grain, entrance tile, balcony concrete, brick facades, and `512×256` equirectangular sky domes (`day` / `sunset` / `night`) so the app renders immediately offline and deploys without external texture failures.
