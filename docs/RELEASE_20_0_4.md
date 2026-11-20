# Kujira 20.0.4: the free cursor and the 20 km leash

Prepared 2026-11-20 in the development copy from installed 20.0.3 (exe 0A05B56B…). The owner played 20.0.3
(screenshot at 10 m cells): past about 500 m to 1 km the view could not pan and kept zooming into one spot; the camera
could not get down to the ground (black sky); the ship was too close to the ground and jittered; and the leash was
misread (the ship was pushed away from the cursor instead of the cursor moving freely around the ship).

Owner rules (2026-11-20):

- The cursor moves freely on screen; held away from the centre it pans the view; the wheel zooms at it.
- The ship stays 20 km above the ground.
- Leash: the cursor moves freely within 20 km of the ship (onto it, above it, below it); only past 20 km does the ship
  follow, at 20 km, while the cursor carries on. The same at every zoom.

Recovery points: `development copy: checkpoints/20.0.4-before-20261120013414` and `GEgine/checkpoints/20.0.4-before-*`.

## Changes (camera, controls and display only)

- **Free cursor** (locked): the software cursor is an ordinary screen pointer (real mouse movement, one pixel per
  pixel, kept inside the window). Held outside the free-flight pan band (140 px from the centre) it pans the view toward
  it with the free-flight gain, so the speed follows the zoom. Panning starts only after the user has moved the mouse
  since the lock; until then the cursor rides on the ship.
- **Zoom at the cursor**: a locked wheel step keeps the world point under the cursor in place (as free zoom does).
  A zoom begun with the cursor by the ship rides down with it (the ship keeps 20 km over the ground through every zoom)
  and, once the ship is higher than a quarter of the view over the ground, holds the ground a quarter of the short side
  below the centre, so zooming in on the ship ends on the ground, never in empty sky.
- **Camera reaches the real ground**: the view point is bounded by the drawn ground under it (minus half a view), not by
  the body's average radius, so valleys below it are reachable (the black screen in the owner's screenshot).
- **20 km floor**: `observation.minimum_standoff_km` 10 → 20 (`gapWorld` = the pixel gap, never under 20 km).
- **20 km leash**: `observation.leash_km` = 20. Only real mouse movement or a pan the cursor holds can pull the ship;
  zoom, easing, window resizes and spin never move it, so it no longer jitters.
- Unchanged from 20.0.3: the rotating frame inside the green boundary, the locked zoom-out limit, no lock at a new
  universe, no ground crosshair. Physics, saves (format 40), the 0.1 m floor and gas-parcel sizes unchanged.

## Tests changed

- 20.0.3's one-point tests replaced: "the cursor moves freely within the 20 km leash; past it the ship follows at 20 km";
  "the ship stays 20 km above the ground; zoom, camera easing and spin never move it"; "a cursor held off centre pans the
  view; over a HUD control the ship never moves"; render: "locked observer: wheel zoom at the ship ends on the ground, the
  free cursor pans, the ship keeps 20 km and its leash, spin moves nothing".
- Re-derived to the free cursor: the v19.4.2 wheel test (the point under the cursor keeps its place), the v19.6.3 locked
  cursor test (the cursor moves by the mouse), v19.1/19.4 framing tests (the surface view frames the observer, on screen).

## Development results (the development copy, pinned)

| Suite | Result |
|---|---|
| Observation module | 84/84 |
| Rendering (ReleaseSafe) | 1,280/1,282, 2 skipped; zoom walk worst step 4.7 (limit 16); full pools p95 ~212 ms at 0.1 m/px |
| Full Debug | 51/51 steps |

## Validation

Final gate `output/20.0.4/validation-final-20261120100550`, every step passed, pinned (affinity FFFF, TEMP in
`GEgine/.zig-cache/tmp`, PC held awake): full Debug 51/51 steps (1,228/1,232 reported, 4 skipped, the rest cached passes of
byte-identical binaries); rendering 1,280/1,282 (2 skipped); full pools p95 207 / 207 ms at zoom 1 / 0.1 m/px (1,280×720),
200 / 207 ms (2,560×1,392), limit 250; BH + 300 replay 96.9 ms; builds 0; workspace integrity 3 as expected. Evidence linked
(`link_evidence_2004.ps1`).

## Installation

Installed 2026-11-20 after the owner's "ALLOW" (`ALLOW 20.0.4 validation-final-20261120100550`). Receipt sealed before the
copy, installed integrity 0, receipt verified after. Installed `ProjectKujira.exe` SHA-256
`009F5834188E93033F881557F68E1EA2B483A8BD2BC387503B874641470D46E9`, byte-identical to the gated build, as are `SDL2.dll` and
`ProjectKujira.pdb`. Rollback to 20.0.3: `checkpoints/20.0.4-before-20261120100550/installed-rollback/`. Player saves were not
opened.
