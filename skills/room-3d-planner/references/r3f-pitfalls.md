# Phase 3/4 Reference: R3F, Drei & Mobile Pitfalls Checklist

Real issues encountered while building 3D apartment simulators and their exact fixes. Check against every item before shipping.

---

### 1. Drei `<Html distanceFactor>` Explodes Under `OrthographicCamera` (`2D 俯視`)
- **Symptom**: Switching to `2D 俯視` causes `<Html>` labels to scale up hundreds of times (`zoom * distanceFactor`) and cover half the screen in solid color blocks.
- **Fix**: Never pass `distanceFactor` to Drei `<Html>` labels that can appear in `OrthographicCamera` views. Keep `<Html position={...} center>` screen-space sized with compact pill CSS (`text-[11px]`).

### 2. `OrbitControls` Binds Stale Camera on Perspective ↔ Orthographic Switch
- **Symptom**: Switching between `3D 鳥瞰` (`PerspectiveCamera`) and `2D 俯視` (`OrthographicCamera`) leaves `OrbitControls` unresponsive or zooming/panning the wrong camera instance.
- **Fix**: Store the camera instance with callback ref state (`const [orthoCam, setOrthoCam] = useState<THREE.OrthographicCamera | null>(null)`) and mount `<OrbitControls makeDefault camera={orthoCam} ... />` conditionally once the camera ref is ready. Reset `.up`, `.position`, `.lookAt`, and `.updateProjectionMatrix()` in `useLayoutEffect`.

### 3. Releasing a Furniture Drag Triggers Canvas `onPointerMissed` / Deselect
- **Symptom**: Dragging furniture to a new spot and releasing the mouse slightly outside or over moving bounds triggers Three.js's `onPointerMissed` and immediately deselects the item.
- **Fix**:
  1. Call `e.stopPropagation()` inside furniture `onClick`.
  2. Track `lastDragEndAt` in Zustand when `setIsDragging(false)` flips from dragging to idle, and ignore `onPointerMissed` if `Date.now() - lastDragEndAt < 350`.

### 4. Clickable Doors/Closets Toggle While Rotating Camera
- **Symptom**: Dragging across a clickable sliding closet door or interior door toggles it open/shut when releasing the mouse button.
- **Fix**: Guard R3F `onClick` with pointer travel distance: `if (e.delta > 4) return;` before `e.stopPropagation()`.

### 5. Unit Mismatch Between `cm` Data and Three.js Meters
- **Symptom**: Straddling clearances or collision offsets fail silently (e.g., checking `height >= 210` against converted meters `h = 2.1`).
- **Fix**: Keep **all** calculations in `src/utils/geometry.ts` and `src/store/` strictly in `cm`; convert to meters (`* CM`, `CM = 0.01`) inside R3F mesh components only, and clearly suffix variables (`heightCm` vs `h`).

### 6. Bay Window Sill vs. Lower Exterior Wall Z-Fighting
- **Symptom**: Modeling a bay-window wooden sill slab (`y = sillY - 3 .. sillY`) overlapping the top of the lower wall (`y = 0 .. sillY`) causes flickering Z-fighting stripes in WebGL.
- **Fix**: End the lower supporting wall slab cleanly at `y2 = sillY - 3` (leaving `-3 cm .. 0` purely for the sill slab).

### 7. CJK Toolbar Labels Wrapping Vertically Inside Flex `overflow-x-auto`
- **Symptom**: Adding controls (like Preview/Edit toggle) to a compact top bar causes CJK button labels (`剖面`, `天花板`, `匯出`) to wrap 1 character per line inside `h-12` flex buttons and clip vertically instead of scrolling horizontally.
- **Fix**: Add `shrink-0 whitespace-nowrap` to **every** flex button inside horizontal toolbars, `shrink-0` on Lucide icons, `min-w-0` on scroll containers, and remove `transition-colors` on instant mode toggles so rapid clicks/screenshots never show blended half-active colors across adjacent pills.

### 8. Multi-Touch Pointer Interference & Sticky Touch Walks/Drags
- **Symptom**: Walking with a virtual joystick while touching the screen to look around drops one finger or leaves the character endlessly walking into walls after `pointercancel` (system gesture, notification shade).
- **Fix**:
  - Track separate `pointerId` variables (`pointerIdRef` on joystick vs. `activeLookPointerId` on canvas).
  - Use window-level `pointermove`, `pointerup`, **and `pointercancel`** listeners matched by `e.pointerId`, wrap `setPointerCapture` in `try/catch`, handle `onLostPointerCapture`, and always call `resetFpVirtualMove()` on unmount.
  - Check `if (e.pointerType === 'mouse' && e.button !== 0) return;` instead of `if (e.button !== 0)` so touch/pen pointers work properly.

### 9. Mobile Viewport Address-Bar Clipping & Page Overscroll Bounce
- **Symptom**: `h-screen` (`100vh`) pushes bottom sheets and joysticks underneath Safari/Chrome mobile navigation bars, and vertical dragging triggers pull-to-refresh.
- **Fix**: Apply `height: 100dvh` (`h-[100dvh]`) on `#root`/layout root, `overscroll-behavior: none` on `html, body, #root`, and `touch-none` on both the Three.js container and the virtual joystick pad.
