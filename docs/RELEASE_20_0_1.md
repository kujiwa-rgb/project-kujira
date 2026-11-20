# Kujira 20.0.1: a plain triangle and the ground in view at deep zoom

Prepared 2026-11-18 in the development copy from installed 20.0.0 (exe C93F29BE…), after the owner played
20.0.0 (2026-11-18, three screenshots): at deep zoom the ground under the triangle was out of view, the triangle changed
shape (hollow, then a pixel outline), and steering a locked planet made the triangle and planet shake. Owner: the
triangle can stay a plain triangle with smooth sides; zoom must let you see the surface and travel along it under the
triangle.
Recovery points: `development copy: checkpoints/20.0.1-before-20261118144555` and `GEgine/checkpoints/20.0.1-before-20261118160254`
(installed 20.0.0 exe, DLL, symbols and the pre-port sources).

## Cause

The triangle is 10 km tall. At 100 m per pixel (the 19.9.12 limit) that is 100 px; at 0.1 m per pixel it is 100,000 px.
20.0 hovered it at its full 10 km above the ground and moved the camera to hold the ground in view, so the triangle sat
far above the top of the screen. The steering centre is the triangle's screen position, so it was off screen; the
software cursor is held inside the window, so every frame read a large deflection, and the view kept being pushed.

## Changes (display and camera only)

- Past 100 m per pixel the drawn triangle keeps its 100 px size (`navigation.triangle_display_zoom_limit` = 3,750,
  `triangleDrawZoom`): the same smooth, anti-aliased triangle as at the 19.9.12 deepest zoom. Up to 100 m per pixel it
  is the true 10 km triangle, unchanged.
- Its hover stand-off uses that drawn size, so it hovers about 24-31 px above the drawn ground at every zoom down to
  0.1 m per pixel, and the 19.9.12 framing (whole triangle kept on screen) applies again at every depth.
- Removed from 20.0: the hollow interior, the 2 px side profile, block sampling and the deep ground hold
  (`observation.deep_ground_fraction`).
- Physics, saves (format 40) and the 0.1 m floor are unchanged. Player saves untouched.

## Tests

- New: "v20.0.1 past 100 m per pixel the drawn triangle keeps its 100 px size".
- Replaced "v20.0 locked deep zoom eases the camera to the ground under the triangle" with "v20.0.1 locked deep zoom keeps
  the triangle and the ground under it on screen, steady while steering" (3,750 to 3,750,000: whole triangle on screen,
  ground close below it, 3 s of steering along the surface with no frame step over 12 px, focus kept).
- "v19.4.2 the triangle hovers above every drawn ground cell" now also runs at 37,500 and 3,750,000.
- The v19.1 true-size test covers zooms up to 3,750 (the true-size range).

## Development results (the development copy, pinned)

| Suite | Result |
|---|---|
| Rendering (ReleaseSafe) | 1,280/1,282, 2 skipped (as 20.0.0); clearance 24-31 px at every zoom; zoom walk worst step 3.5 (limit 16) |
| Full Debug | 51/51 steps, 1,440/1,445, 5 skipped |

## Validation

Final gate `output/20.0.1/validation-final-20261118160254`, every step passed, pinned (affinity FFFF, TEMP in
`GEgine/.zig-cache/tmp`).

| Step | Result |
|---|---|
| Focused steps, completion, probes, launcher guard | all 0 |
| Full Debug | 51/51 steps, 1,440/1,445 passed, 5 skipped |
| Rendering (ReleaseSafe) | 1,280/1,282 passed, 2 skipped |
| Full pools p95 (1,280x720 / 2,560x1,392) | 200 / 201 ms at zoom 1; 215 / 216 ms at 0.1 m/px; limit 250 |
| BH + 300 cluster replay | worst play frame 88.4 ms (limit 250) |
| Debug and ReleaseSafe builds | 0 |
| Workspace integrity | 3, as expected |

Acceptance evidence linked (`link_evidence_2001.ps1`); the seal check
(`gate-receipt-preauthorization.log`) is blocked only by the missing owner authorization.

## Installation

Installed 2026-11-19 after the owner's "ALLOW" in the thread (written as
`ALLOW 20.0.1 validation-final-20261118160254` in `output/install-authorized.txt`). The receipt was sealed before the copy
(`gate-receipt-final.log`, `install-final.log`), the installed integrity check returned 0
(`integrity-installed-final.log`), and the receipt verified again after installing (`receipt-postinstall.log`).
Installed `ProjectKujira.exe` SHA-256: `364A00C41F802E859F9E2930363927720DA5DF8F2553AA40F693818BE1B36BFC`, byte-identical
to the gated build, as are `SDL2.dll` and `ProjectKujira.pdb`. Rollback to 20.0.0:
`checkpoints/20.0.1-before-20261118160254/installed-rollback/`. Player saves were not opened.
