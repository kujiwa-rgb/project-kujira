# Project Kujira: current development stage (v20.0)

As of **November 20, 2026**. Public documentation update only.

## Installed release

- **20.0.4** is installed: the first v20 version (20.0) and its four owner-tested corrections are complete.
- Save format **40** (unchanged since 19.9.12).
- Installed executable SHA-256: `009F5834188E93033F881557F68E1EA2B483A8BD2BC387503B874641470D46E9`.
- Final gate: `output/20.0.4/validation-final-20261120100550/`. Every step passed; the receipt was sealed before the copy and
  verified after it; installed integrity returned **0**; executable, SDL2 and symbols match the gated build.
- The owner played 20.0.4 and confirmed the locked observer works as expected (2026-11-20).

These are recorded development results. No engine test, build or installation was performed for this documentation update.

## What v20.0 changed

**20.0.0, the 0.1 m floor.** One size floor for every solid and every gas parcel: **0.1 m (1 dm)**, replacing the 1 km
floor everywhere it applied (births, fragments, debris, packets and bins, creation, cleanup, focus and display). Zoom
reaches **0.1 m per pixel**, so the floor is exactly one pixel at the closest zoom. Small solids now merge or finish
evaporating into a gas packet (nothing is deleted); free evaporation follows the Hertz-Knudsen limit per second, not per
step; stellar wind and planetary venting scale down to the same floor.

**20.0.1-20.0.4, the observer at deep zoom** (four owner play tests):

- The triangle is always the true **10 km** triangle: never resized, never faded, smooth edges at any size.
- Locking parks the observer in the planet's rotating frame inside its **green boundary**, so the planet holds still.
- The cursor is free: held off centre it **pans** the view (speed follows the zoom); the wheel **zooms at the cursor**.
- Zooming in on the ship rides down with it and ends on the **ground** under it; the camera reaches valleys.
- The ship keeps **20 km** over the ground and trails the cursor on a **20 km leash**: the cursor moves freely within
  20 km of it, and only past that does the ship follow.
- Locked, zooming out stops at the green boundary; a new universe does not lock by itself.
- The HUD scale reads down to **10 m**; gas parcels draw at their own size like solids.

## Gate evidence (20.0.4)

| Step | Result |
|---|---|
| Focused steps, completion, probes, launcher guard | all passed |
| Full Debug | 51/51 steps; 1,228 of 1,232 reported tests passed, 4 skipped; the rest cached passes of byte-identical binaries |
| Rendering (ReleaseSafe) | 1,280 of 1,282 passed, 2 skipped |
| Full pools frame p95 (1,280x720) | 207 ms at zoom 1, 207 ms at 0.1 m per pixel; limit 250 ms |
| Full pools frame p95 (2,560x1,392) | 200 ms at zoom 1, 207 ms at 0.1 m per pixel |
| Black hole + 300 star replay | worst play frame 96.9 ms (limit 250) |

The gate ran pinned to the 16 performance-core threads of the i7-14700F with the PC held awake. Skips are not passes, and
cached execution is not described as fresh.

## Declared limits

- Positions are f64 world units of 3,000 km: a 0.1 m body is placed exactly only within a few hundred AU of the origin;
  anchored positions are planned.
- Persistent metre-scale terrain is v20.2; at 0.1 m per pixel a planet shows its existing surface fields at finer cells.
- Waves driven by gas currents are planned for v20.3.

## Next: v20.1

The owner's add-ons from the 20.0.4 play test: a flatter ground line at surface zoom that follows the body's true curvature
plus real relief only (decided), and gas-cloud outlines that refine with zoom toward fluid surfaces. Crater-driven surface
topologies by size and mass follow with v20.2 terrain.
