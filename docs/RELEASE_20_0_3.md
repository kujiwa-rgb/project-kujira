# Kujira 20.0.3: the locked "UFO" observer

Prepared 2026-11-19 in the development copy from installed 20.0.2 (exe C76E01F0…). The owner tested 20.0.2 and
found the locked deep zoom broken: the ship filled the screen when looking up, the view shook, two marks did nothing, the
grid turned as the camera chased the triangle. Design and the owner's answers: `docs/V20_0_3_OBSERVER_MODEL.md`.

Owner rules (2026-11-19):

- Locking parks the ship (the triangle) in the planet's rotating frame inside its green boundary: the planet holds still
  under it, and the grid is seen turning.
- The ship keeps 10 km from the ground. Zoomed in past where it fits the view, the mouse pans the view over the surface
  and the air and the ship follows like a dog on a leash, within 10 km, never entering the view; zooming back out
  returns smoothly to flying the ship.
- Speed at scale: panning moves the view one screen pixel per pixel of mouse, so 100 px is 100 m at 1 m per pixel and
  thousands of kilometres at planet scale.
- While locked, zooming out stops at the green boundary.
- A new universe does not lock onto a body by itself (hovering still locks once the mouse has moved).

Recovery points: `development copy: checkpoints/20.0.3-before-20261119153908` and `GEgine/checkpoints/20.0.3-before-*`.

## Changes (camera, controls and display only)

- **Rotating frame**: the locked view point C (body-fixed angle, height above the body's base radius) and the ship are
  kept in the body's frame, so spin and orbit are taken out exactly (no camera easing). Outside the green boundary
  (`physics.planetaryApproachRadius`, passed in `Body.limit_radius`; four radii where none) the observer keeps its own
  world direction. The side-on turn follows C exactly; only a new lock eases in.
- **One point under the mouse**: sideways movement moves C along the surface; up and down move the ship while it fits
  the view and the view itself once it has outgrown it (`under_weight`, 25 % to 100 % of the short side). One mouse pixel
  is one screen pixel.
- **Ground in view**: left alone, the view holds the ground under C a quarter of the short side below its centre once
  the ship is higher than that (`ground_hold_fraction`), so zooming never takes the ground out of view.
- **Leash**: once the ship outgrows the view it stays sideways within the leash (10 km nose to C), above C by at least
  the view's half-diagonal plus its own reach (never in view), at most the leash above C, never under its 10 km floor or
  past the green boundary. Where its place is clear it is placed at once (a lagging ship could cross the view).
- **10 km floor**: unchanged from 20.0.2 (`gapWorld` = the pixel gap, never under 10 km); ground relief under the
  footprint lifts the floor at once and lets it settle back gently.
- **Zoom**: locked, the wheel zooms about the view (no anchor jump); zooming out stops where the green boundary fills
  the short side with a quarter to spare (`State.lockedMinimumZoom`).
- **Marks**: the ground crosshair is removed; the one mark is the cursor, which rides on the ship (or on C once the ship
  has outgrown the view) and leaves it only by movement it cannot follow.
- **No lock at a new universe**: `resetUniverse` and ADD STAR SYSTEM start with `pick_armed = false` until the mouse moves.
- Physics, saves (format 40), the 0.1 m floor and the gas-parcel sizes of 20.0.2 are unchanged. Player saves untouched.

## Tests changed

- Joystick-era steering tests re-derived for the one-point model (axes, one-to-one at any frame rate, bounds by the green
  boundary and the 10 km floor, cursor overflow, menus and resize).
- New: "v20.0.3 locked UFO observer: planet held still, ship out of view at deep zoom, free pan within bounds, smooth zoom
  back" (37,500 to 3,750,000: ship outside the view and within the leash, ground in view, 200 px of panning moves the view
  200 px, spin moves nothing, zoom-out steps move the view under 100 px).
- New: "v20.0.3 a new universe does not lock by itself, and a locked view zooms out no farther than the green boundary".
- v19.4.2 wheel step: the ship may move by the step's share of its distance from the centre once the ground hold holds.

## Development results (the development copy, pinned)

| Suite | Result |
|---|---|
| Rendering (ReleaseSafe) | 1,283/1,285, 2 skipped; zoom walk worst step 4.0 (limit 16); full pools p95 196 / 204 ms at zoom 1 / 0.1 m/px (1,280×720), 195 / 203 ms (2,560×1,392); BH+300 85.3 ms |
| Observation module | 87/87 |
| Full Debug | 51/51 steps |

## Validation

Final gate `output/20.0.3/validation-final-20261119173634`, every step passed, pinned (affinity FFFF, TEMP in
`GEgine/.zig-cache/tmp`, PC held awake).

| Step | Result |
|---|---|
| Focused steps, completion, probes, launcher guard | all 0 |
| Full Debug | 51/51 steps, 1,228/1,232 reported (4 skipped), the rest cached passes of byte-identical binaries |
| Rendering (ReleaseSafe) | 1,283/1,285 passed, 2 skipped |
| Full pools p95 (1,280x720 / 2,560x1,392) | 199 / 200 ms at zoom 1; 208 / 205 ms at 0.1 m/px; limit 250 |
| BH + 300 cluster replay | worst play frame 92.8 ms (limit 250) |
| Debug and ReleaseSafe builds | 0 |
| Workspace integrity | 3, as expected |

Evidence linked (`link_evidence_2003.ps1`).

## Installation

Installed 2026-11-20 after the owner's "ALLOW" (`ALLOW 20.0.3 validation-final-20261119173634`). Receipt sealed before
the copy, installed integrity 0, receipt verified after. Installed `ProjectKujira.exe` SHA-256
`0A05B56BD5B6D28FF3410FFE5A52BE0FAC54F6D32476B6E24811830EB16B6411`, byte-identical to the gated build, as are `SDL2.dll` and
`ProjectKujira.pdb`. Rollback to 20.0.2: `checkpoints/20.0.3-before-20261119173634/installed-rollback/`. Player saves were
not opened.
