# Kujira 20.0.2: the true triangle, the ground in view and free movement at deep zoom

Prepared 2026-11-18 to 2026-11-19 in the development copy from installed 20.0.1 (exe 364A00C4…). The owner
played 20.0.1 and rejected it: capping the triangle's drawn size past 100 m per pixel made it change scale against the
world ("a big no-no"). Owner rules for deep zoom (2026-11-18/19):

- The triangle is always the true 10 km triangle, at every zoom, with smooth edges (never pixel-stepped), fully opaque.
- The surface under it is in view while zooming, down to 0.1 m per pixel.
- The user can move anywhere around a locked planet with the mouse, at a speed set by the zoom (owner choice A,
  "free look inside the lock").
- The triangle keeps a limit distance from the surface (at least 10 km), so it never covers what is under it in view.
- The HUD scale reading goes below 10,000 km.

Recovery points: `development copy: checkpoints/20.0.2-before-20261118183143` (development) and `GEgine/checkpoints/20.0.2-before-*`
(installed 20.0.1 exe, DLL, symbols and the pre-port sources).

## Changes (display, camera and controls only)

- **True triangle**: the 20.0.1 size cap is removed (`navigation.triangleRasterRadiusPixels` and `triangleHeightPixels`
  use the true scale again). Up to 192 px of radius it is drawn by the 16-sample pixel sampler as before; above that by
  its exact fill and outline-band polygons (`navigation.triangleBandCorners`, the same half-planes and colours) with a
  1 px coverage ramp on every edge, so it is smooth at any size without the ~130 ms per-pixel cost.
- **10 km stand-off**: `observation.gapWorld` is the v19.1 pixel share of the short side, but never under 10 km
  (`minimum_standoff_km`). The nose stays at least 10 km above the highest drawn ground under the triangle's footprint.
  Up to about 0.5 km per pixel the pixel share is larger, so nothing changes there.
- **Ground in view**: the locked surface view frames the triangle until its centre is a quarter of the short side above
  the ground spot under its nose; past that the ground spot stays a quarter of the short side below the view centre
  (`observation.ground_hold_fraction`), so the view looks under the triangle. The hand-over is continuous (zoom walk
  palette step 4.0, limit 16). A crosshair marks the ground spot once the triangle is a quarter of the view tall.
- **Mouse moves the triangle directly** (replaces the v19.5 joystick): each real mouse movement moves the triangle the
  same screen distance in the displayed axes, along the surface and above it (up to the v19.5 altitude bound, never into
  the body). The view follows it, so moving the mouse travels anywhere around the planet; at 0.1 m per pixel one pixel of
  movement is 0.1 m of travel. Movement the triangle cannot follow (past the altitude bound, into the ground) moves the
  software cursor away from it, so the UNLOCK FOCUS button stays reachable; moving back first returns the cursor. Under
  a triangle taller than a quarter of the view, up/down looks up toward the triangle or back down to the ground.
- **Zoom**: a focused wheel step keeps the triangle in place, or, under a big triangle, the spot the view is framing.
- **Unlocked at deep zoom**: the true triangle sits on the pointer while it is small; as it grows from a quarter to
  the whole of the view it rises until its nose points down at the pointer from 10 km above
  (`main.freeTrianglePlacement`, display only), so the spot under the pointer stays in view (it filled the screen).
- **Gas parcels at true size** (owner, 2026-11-19): packet and bin gas samples draw at their parcel size like solids,
  scaling with the zoom down to the 0.1 m floor (they were always one pixel); below one pixel a parcel is one pixel.
  Waves driven by gas currents are planned in `V20_3_PLAN.md` (decision D19).
- **HUD scale**: `topology.hudScaleForZoom` continues the 1-2-5 ladder past the universe ceiling with the same 64 px
  minimum, down to 10 m at 0.1 m per pixel ("CELL 10.0 M / …"). The drawn grid is unchanged.
- Physics, saves (format 40) and the 0.1 m floor are unchanged. Player saves untouched.

## Tests changed (superseded contracts)

- v19.5/19.6.1/19.6.3 joystick tests (dead zone, bounded speed, smooth stop, viewport-scaled sensitivity, joystick
  altitude) re-derived for direct movement: "mouse movement moves the triangle along and above the surface in the
  displayed axes"; "the triangle follows mouse movement one to one, with no dead zone, at any frame rate"; "altitude is
  bounded and clear; the cursor keeps what the triangle cannot follow; menus and resize leave no stale input";
  "movement is the same screen distance on any window size and a resize clears pending input".
- 20.0.1's deep test replaced by "v20.0.2 under a triangle taller than the view the ground spot stays in view, the mouse
  moves along the surface and zoom anchors on the view" (37,500 to 3,750,000: ground spot a quarter below centre, nose at
  least 10 km up, 200 px of movement travels 200 px along the surface, no screen jumps, focus kept, zoom anchored).
- The v19.1 true-size test runs at every zoom up to 3,750,000 again.
- New: "the triangle's band corners match the pixel sampler's outline and fill"; "HUD scale follows the zoom down to
  0.1 m per pixel"; "gas parcels draw at their own size and scale with the zoom down to the 0.1 m floor" (a 1 kg cloud of
  1 m parcels: about 1-3 px at 1 m per pixel, 10 px parcels at 0.1 m per pixel, the triangle not covering the view).
- Triangle opacity is never changed (an interim 25 % fade was removed on the owner's word).
- The v19.4.2 zoom walk measures the surface palette with the triangle's pixels left out (the true triangle now grows
  over the view past 100 m per pixel).

## Development results (the development copy, pinned)

| Suite | Result |
|---|---|
| Rendering (ReleaseSafe) | 1,282/1,284, 2 skipped (as 20.0.1); zoom walk worst step 4.0 (limit 16); full pools p95 195 / 201 ms at zoom 1 / 0.1 m/px (1,280×720), 197 / 208 ms (2,560×1,392), limit 250; BH+300 replay 84.4 ms |
| Observation module | 87/87 |
| Full Debug | 51/51 steps |

## Validation

First gate `output/20.0.2/validation-final-20261118214741` void: the PC idle-slept at 22:27 and a REBOUND test runner
timed out on waking (every step before that passed). Final gate `output/20.0.2/validation-final-20261119075016`, every
step passed, pinned (affinity FFFF, TEMP in `GEgine/.zig-cache/tmp`, PC held awake).

| Step | Result |
|---|---|
| Focused steps, completion, probes, launcher guard | all 0 |
| Full Debug | 51/51 steps, 1,116/1,120 reported (4 skipped), the rest cached passes of byte-identical binaries |
| Rendering (ReleaseSafe) | 1,282/1,284 passed, 2 skipped |
| Full pools p95 (1,280x720 / 2,560x1,392) | 202 / 203 ms at zoom 1; 213 / 206 ms at 0.1 m/px; limit 250 |
| BH + 300 cluster replay | worst play frame 90.7 ms (limit 250) |
| Debug and ReleaseSafe builds | 0 |
| Workspace integrity | 3, as expected |

Evidence linked (`link_evidence_2002.ps1`).

## Installation

Installed 2026-11-19 after the owner's "ALLOW" (`ALLOW 20.0.2 validation-final-20261119075016` in
`output/install-authorized.txt`). Receipt sealed before the copy, installed integrity 0, receipt verified after.
Installed `ProjectKujira.exe` SHA-256 `C76E01F01A978367E2DC254A65DA927E2CA49D514CD1D329DC26453353EB6C66`, byte-identical to the
gated build, as are `SDL2.dll` and `ProjectKujira.pdb`. Rollback to 20.0.1:
`checkpoints/20.0.2-before-20261118214741/installed-rollback/`. Player saves were not opened.
