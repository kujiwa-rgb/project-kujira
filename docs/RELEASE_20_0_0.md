# Kujira 20.0.0: the 0.1 m floor, zoom to 0.1 m per pixel, and the regression behind them

Prepared 2026-11-06 to 2026-11-18 in the development copy from installed 19.9.12 (exe 2B5E5AE1…),
to `docs/V20_PLAN.md` (revision 7) and `docs/V20_0_PLAN.md`, following the owner's process (section V.4): built in the
development copy, ported to `GEgine` byte-identical, then gated. Owner go: "go 20.0 with recommendations" (2026-11-06),
with D14 yes, D16 measure first.
Recovery points: `development copy: checkpoints/20.0-before-20261106100139` (development) and `GEgine/checkpoints/20.0-before-*`
(installed 19.9.12 exe, DLL, symbols and the pre-port sources).

## Owner rules this version implements

- **One floor for every solid and every gas parcel: 0.1 m (1 dm)** (2026-11-01, final for v20). It replaces the 1 km
  floor everywhere it applied: births, fragments, debris, packet and bin constituents, creation, the sub-floor cleanup,
  focus and display. No larger v20 floor remains.
- **Zoom reaches 0.1 m per pixel**, so the floor is exactly one pixel at the closest zoom.
- **Gas scales down with solids** (2026-11-07): stellar wind from small sources and planetary venting go down to 0.1 m.
- **Small solids merge or evaporate** (2026-11-10): nothing under the floor lingers or is deleted.
- **Evaporation per second, not per step**, and Stellar Time is not forced to resolve tiny bodies' fast details.
- **The triangle at the deepest zoom** (2026-11-13): side profile instead of blocking the view; a locked camera keeps the
  ground under the triangle in view; the hand-over is smooth.

## Changes

### Floor and debris (physics)

- `material_packets.minimum_constituent_m` 1,000 m → **0.1 m**. Every bare "1000" that meant the floor now reads this one
  constant (behaviour-neutral step, checked 1,097/1,097 before the value changed).
- Curtain draw floor `curtain_minimum_m` 2 km → **0.2 m**: the same factor of two above the floor that 19.9.1 used
  (a freshly broken piece compacts 13–24 % and must stay above the floor). The 19.9.1 fragment tail `N(>d) ~ d^-2.85`,
  piece budget, speed law and paid escape are unchanged; the tail now reaches 0.2 m. Mass beyond the piece budget stays in
  the donor (the 19.9.1 rule, not a size limit).
- The ejecta split's tail reference follows the floor (`massFractionAbove(375 km, floor, d_max)`).
- Since 19.6.4 solids cannot be packet distributions; that rule is unchanged.

### Creation and focus (camera)

- Creation minimum 1 km → 0.1 m for every material; the size keys continue below 1 km (500, 200, 100, 50, 20, 10, 5, 2,
  1, 0.5, 0.2, 0.1 m).
- Lock and focus down to 0.1 m (`observation.minimum_diameter_km` = floor). The 19.9.11 obstacle rule is unchanged, so
  small bodies never all become steering obstacles.

### Small-body physics

- **Free evaporation limit (Hertz–Knudsen)**: `material_phase.freeEvaporationFlux`, J = P_sat(T) sqrt(M / 2 pi R T) with
  evaporation coefficient 1 (an upper bound). `planetary.phasePlan` evaporation can never exceed that flux times the
  sector's area and the interval. Planets never reach the limit (unchanged); small hot bodies do, and their loss now scales
  with time instead of with the number of steps. Cause found: the saturation "vapour column" of a body with almost no
  gravity exceeds its own mass, and the old per-call cap of half the available inventory made the result depend on how an
  interval was divided, so detailed Stellar Time could not advance a hot 0.1 m drop (steps fell to ~1e-11 s).
- **Sub-floor solids** (`cleanupUndersizedSolids`, decision D14): nothing is deleted any more. At real contact they merge as
  before. An isolated one finishes evaporating (`planetary.finalEvaporationPlan`, `Simulation.evaporateUndersized`) when
  its own heat plus the sensible heat of its vapour pays the latent heat of its last condensed Irio and Magnesio; it then
  leaves the body table as a gas packet at least 0.1 m across, with mass, momentum, heat and charge-free inventory kept, and
  its pair potentials re-booked into the packet's binding. Otherwise it stays a body and merges at contact.
  `undersized_culled_kg` stays zero.

### Gas below 375 km

- Stellar wind (`wind.packetPlan`): a source too small for the v16.4 375 km sampled range emits parcels from the 0.1 m gas
  floor up to its own ceiling instead of none. Large sources keep their sizes, mass loss and parcel counts.
- Planetary venting: parcels down to the 0.1 m floor (was 375 km).
- The 375 km resolved-fragment size stays the boundary between resolved chips and the sub-floor curtain, and the merge rule
  for sub-375 km gas remnants captured by a planet is unchanged.

### Zoom and drawing (display only)

- `topology.surface_maximum_zoom` 3,750 (100 m/px) → **3,750,000 (0.1 m/px)**.
- Rocks below one surface cell are drawn as an area-equivalent square at true size: a 0.1 m rock is one pixel at 0.1 m/px,
  a 1 m rock about 10 px, near the origin and 5.2 light years out (drawing is camera-relative).
- Surface cells keep halving at every doubling of zoom past 100 m/px (`raster.v19_surface_zoom`), so they stay 15–30 px on
  screen down to 0.1 m/px and the same surface fields show finer; unchanged up to 100 m/px.
- `raster.halfColumns` saturates (a star at 0.1 m/px is ~1e10 cells across; spans were already clipped).
- The 10 km triangle: above 192 px of radius its coverage is sampled in blocks (per-pixel coverage cost ~130 ms a frame at
  0.1 m/px). From a tenth of the short side upward its interior opens from the centre, the remaining ring narrowing by the
  same share each zoom step to the 2 px **side profile**; the profile is drawn while it opens. The triangle stays
  physically 10 km.
- Locked deep zoom: once the triangle is a quarter of the short side tall, the camera's framing target keeps the ground
  under the triangle within a quarter of the short side below the centre, so locking and relocking still ease into it.
  Below that size framing is exactly as in 19.9.12.

Saves: format 40, unchanged (new runtime counters are not saved). Player saves untouched.

## Tests changed to the owner's rules (superseded contracts)

- 1 km floor tests re-derived to 0.1 m: the v19.6.4 size-floor, hard-solid, conversion, unified-solids and cutoff tests,
  the v19.6.4 deep-time cutoff test, the v19.9.2 creation tests, the material-packet size law and encounter regimes, the
  two-jet seam test, the v19.6.1/19.6.2 impact audits' smallest piece, the creation carousel walks.
- D14: "cutoff retires isolated owner" → "keeps an isolated undersized owner"; "merges contact, never across a gap" →
  "merges contact and keeps a separated owner"; new "isolated undersized hot drop finishes evaporating into a gas packet,
  conserving everything" (mass, energy, momentum, centre of mass and angular momentum to 1e-12).
- Deep time: "ordinary, coarse and detailed skips keep a phase-shrinking body at the 0.1 m floor"; detailed mode (the
  sub-100-year reference) gets the work to finish (462 steps) instead of a looser tolerance; coarse mode takes one step.
- Wind: the giant-wind test now expects a 1,000 km source to emit a parcel between the gas floor and 40 km; new
  "a source too small for the 375 km spread emits wind down to the 0.1 m gas floor".
- Zoom: the surface-range test expects 0.1 m/px; the wheel test uses 80 clicks; new "0.1 m floor is one pixel at 0.1 m per
  pixel and small rocks draw at their true size"; new "surface cells keep a bounded screen size"; new "locked deep zoom
  keeps the ground under the triangle on screen"; the v19.4.2 zoom walk now walks to 0.1 m/px (worst step 5.9, limit 16).
- New: Hertz–Knudsen flux test.

## Development results (the development copy, pinned to the performance cores)

| Suite | Result |
|---|---|
| Physics (ReleaseSafe) | 1,100/1,100 |
| Rendering (ReleaseSafe) | 1,279/1,281, 2 skipped (as 19.9.12) |
| Full Debug | 51/51 steps, 1,438/1,443, 5 skipped |
| Full pools live frame p95 | 200 ms at zoom 1, 205 ms at 0.1 m/px (1,280×720); 196 / 206 ms (2,560×1,392); limit 250 |

## Known and deferred

- D16 precision: positions are f64 world units of 3,000 km; the step is 2e-5 m at 1 AU, 0.09 m at 0.05 ly, 1.4 m at 1 ly,
  5.7 m at 5.2 ly. A 0.1 m body is placed exactly only within a few hundred AU of the origin; anchored positions are the
  recommended follow-up.
- Persistent metre-scale terrain is 20.2; at 0.1 m/px the planet shows its existing surface fields at finer cells.
- The engine has no metal-vapour gas body of its own; an evaporating drop keeps its vapour on itself until the last step.
- Lean L1–L3 need the owner's separate yes to install the toolchain.

## Validation

Final gate `output/20.0.0/validation-final-20261105105748` (the run name repeats 19.9.12's because `next-run.txt`
was not rewritten; it lives under the 20.0.0 folder). Every step passed. It ran with processor affinity FFFF (the 16
performance-core threads of the i7-14700F) and TEMP set to `GEgine/.zig-cache/tmp`, with nothing else running.

| Step | Result | Fresh or cached |
|---|---|---|
| Focused steps 19.6.2-19.9 (34 steps) | all 0 | ran |
| Populated completion, cooling, probes, cluster benchmark | 0 | ran |
| Launcher guard | 0 | ran |
| Full Debug | 51/51 steps, 1,438/1,443 passed, 5 skipped | 29 test programs ran fresh (physics 1,096/1,100, 4 skipped); 3 cached passes of byte-identical binaries |
| Rendering (ReleaseSafe) | 1,279/1,281 passed, 2 skipped | ran |
| Full pools p95 (1,280×720 / 2,560×1,392) | 200 / 200 ms at zoom 1; 213 / 210 ms at 0.1 m/px; limit 250 | ran |
| BH + 300 cluster replay | worst play frame 89.8 ms (limit 250) | ran |
| Debug and ReleaseSafe builds | 0 | ran |
| Workspace integrity | 3, as expected | ran |

Acceptance evidence linked (`link_evidence_2000.ps1`); the seal check is saved as
`gate-receipt-preauthorization.log` and is blocked only by the missing owner authorization.

Recovery point: `checkpoints/20.0-before-20261117193436` (installed 19.9.12 exe SHA-256
`2B5E5AE1AC50A37D6FA7E990D5998CED59CC122F44A3A531E5854ED8CEECB569`, SDL2.dll, symbols, and the pre-port sources).

## Installation

Installed 2026-11-18 after the owner's "ALLOW" in the thread (the only pending candidate, written as
`ALLOW 20.0.0 validation-final-20261105105748` in `output/install-authorized.txt`). The receipt was sealed and verified
before the copy (`gate-receipt-final.log`, `install-final.log`), the installed integrity check returned 0
(`integrity-installed-final.log`), and the receipt verified again after installing (`receipt-postinstall.log`).
Installed `ProjectKujira.exe` SHA-256: `C93F29BE07F1DBF2A0391D98AECE380B808B59FB0472A31A3CFBF035A1904AEF`, byte-identical
to the gated build, as are `SDL2.dll` and `ProjectKujira.pdb`. Rollback to 19.9.12:
`checkpoints/20.0-before-20261117193436/installed-rollback/`. Player saves were not opened.
