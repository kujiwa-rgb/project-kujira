> Public documentation snapshot, October 29, 2026. Engine source and local gate/rollback files remain private. Results below are recorded development evidence, not tests executed for this publication.

# Kujira 19.9.0: impact debris, planetary vapour, water display and Roche check

Prepared 2026-10-28 in `the private development checkout` from installed 19.8.1, from the owner's
four reports (screenshots 2026-10-28 09:30-09:38). 19.9 is the v19 slot reserved for fixes.
Save format 37 is unchanged. Gravity, contact detection, thermal laws, budgets and capacities
are unchanged.

## 1. Equal-size impact debris (owner: less exhaustive; directional from the contact side)

Reproduced: two equal 4,500-km Irio bodies at 10 km/s near a 0.29 Msun star made 66 solid
debris bodies at once. Some were placed around the whole circumference, and the resolved
recoil chips were pushed through the partner to its far side. Repeated impacts filled the
1,024 main-body slots ("MATTER 924/1024").

Three placement rules changed. Mass, momentum, angular momentum, energy and the 1 km minimum
are still settled by the same transactions.

- **Sub-floor curtain** (`emitDebrisCurtain`):
  - At most 8 pieces per curtain (`curtain_piece_limit`), each the representative
    constituent as before. Budgeted mass beyond them stays in the donor; nothing is enlarged.
    A first version with enlarged pieces was rejected by the gate: equal-merger event energy
    drifted to 1.3e-4 against the 1e-4 limit, and one piece left 0.15 % above escape.
  - Each piece is placed inside a 40-degree cone about the limb of the contact seam. A merged
    pair uses both limbs, perpendicular to the impact axis. A surviving donor leans 45 degrees
    toward its own contact side.
  - Debris never wraps around the body. A piece that cannot be placed stays with its donor.
- **Resolved recoil chips** (`trySheddingPlan`, solid recoil only): chips squeeze out of the
  seam sideways, alternating the two limbs, opposite for the two donors. They are no longer
  pushed along the contact normal through the partner.
- **Ring ejecta** (`emitPlanetaryRing`): a merger's ring fragments start at the two seam limbs
  (within +-15 degrees) instead of evenly around the planet. Orbital motion spreads them
  afterwards.

Result in the same scene: 12 debris bodies instead of 66, all in two opposite jets along the
seam; all of them fall back within 3,000 ticks. Test: "v19.9 equal impact ejecta leave as two
opposite bounded jets from the contact seam".

Tests changed to the owner's new rules:
- **Hard-solid tests (three):** they asserted one body per requested constituent; they now
  expect at most 8 pieces of the same size.
- **"v19.6.2 shared-body solid ejecta is paid once":** it required sub-floor packets; now the
  sideways recoil chips can take the whole loss budget, so it requires real ejecta of either
  kind. Its conservation and cost checks are unchanged.
- **"v19.5d clouds draw as pixel layers":** it keeps its contract. Clouds brighten only the
  ring between the ground and the drawn top, and drawing changes nothing.

## 2. Star-like gas debris appearing around a planet (owner: free gas debris comes from stars)

Reproduced: a hot planet (core above 30,000 K after an impact) vented hydrogen and oxygen from
its own steam as a new main body. It was labelled "HYDRIO GAS DEBRIS" and drawn as a glowing
gas ball beside the planet.

Thermal venting (`emitPlanetaryAtmospheres`, not Roche overflow) now leaves as a gas packet in
the shared pool, drawn as a thin vapour cloud. It still falls back or escapes under the same
gravity and drag. The transaction is unchanged:
- the same mass, heat and slow launch;
- the remnant's potential is re-booked;
- packet-packet gravity is omitted, as for every packet.

Roche-lobe overflow from gas-rich planets toward their star stays a body. Stellar winds and
stellar debris remain the only free gas bodies near stars. Three vent tests now find the
vented gas in the packet pool. Their mass, species and ledger checks are unchanged.

## 3. Water, ice and vapour display (rendering only)

Reproduced at the owner's state: a 4,500-km iron body with 0.02 % water. A trace atmosphere
(about 1e16 kg of vapour) was drawn over the full 750-km envelope, and clouds were drawn as
375-km universe cells:
- an opaque white ring around the planet, with a black gap between the ring and the ground;
- a dotted ring with gaps along the arc (at most 16 squares per column);
- at deep zoom, the whole view covered by blue-white blocks, hiding the refined surface.

Changes (`main.zig`, `artifacts.zig`), reviewed with the owner on captures. Display only: no
simulation state is written, and the envelope, gravity and every budget are unchanged.

- **Clouds and vapour are part of the one gas layer.** Separate cloud cells are no longer drawn.
  The body's cloud water (mean opacity of its 64 columns from their optical depth) joins the
  gas layer's colour and opacity (`cloudSummary`, `withClouds`). They use the same raster,
  cells and radial gradient as any atmosphere over a solid.
- **Vapour air** (water, rock or metal vapour at least half the retained gas, or cloud water
  only) is drawn:
  - on the planet's own cells, so its pixelation and outline match the surface;
  - behind the planet only: it shows past the limb and never over the surface;
  - with its side-on cross-section rising from the drawn ground at deep zoom.
- **Vapour height follows temperature.** It is one scale height of the air,
  `H = R T / (mu g)`, times its column share (`clamp(column / 1e3 kg/m^2, 0.1, 1)`), and never
  more than the envelope. Captures: a 2,288 C molten planet shows about 120 km of steam
  (envelope 750 km); temperate and cold planets about 4 km and 2 km. Below half a planet
  pixel the layer is not drawn at that zoom, so it never becomes a whole-cell ring.
- **Other trace atmospheres** are drawn in proportion to their column,
  `clamp(sqrt(column / 1e4 kg/m^2), 0.02, 1)` of the envelope. An Earth-like column or more
  keeps the whole envelope, and the ring is at least one whole cell, without gaps.
- **Colours** (owner):
  - water vapour is a pale blue made from the water blue (60 % toward white) at low opacity,
    at most 0.35 and scaled by its column;
  - cloud water uses the same pale blue (ice slightly lighter), at most 0.60;
  - surface ice is light blue (0.62, 0.82, 0.98), and open water keeps its ocean blue.

Captures: `v199 capture` (run with `KUJIRA_CAPTURE=1`), cold, temperate and molten 4,500-km
planets at zoom 1-1536.

## 4. Stellar Roche zone and winds (checked, no regression in the mechanism)

- A lone 0.29 Msun star in game settings emits bursts of three winds. Over 40,000 ticks
  (4.6 days), 5 returning winds broke up inside the tidal zone, as before.
- The same star with 300 extra bodies broke up its first wind at the same tick.
- In the owner's scene, about 900 impact debris bodies held the table. The shared fragment
  budget then allows one new piece per 8 ticks, and debris contacts compete for it, which
  starves wind break-up and emission (inferred from the code and the measured counts). Fix 1
  removes that flood.

## Validation

Final gate `output/19.9.0/validation-final-20261028214627`. Every step passed. It ran with
processor affinity FFFF, the 16 performance-core threads of the i7-14700F (see below).

| Step | Result | Fresh or cached |
|---|---|---|
| Focused steps (all, including `focused-19.9-physics` with the full-table vent test) | 0 | ran |
| Launcher guard | 0 | ran |
| Full Debug | 51/51 steps | 21 test programs reused cached passes |
| Rendering (ReleaseSafe) | 1,240/1,242 passed, 2 skipped | ran |
| BH + 300 cluster replay | worst play frame 91.1 ms (limit 250) | ran |
| Populated completion | 9/9, linked REBOUND | ran (it always runs fresh) |
| Debug and ReleaseSafe builds | 0 | ran |
| Workspace integrity | 3, as expected | ran |

The full-suite cache reuse is genuine. The source is byte-identical to the previous gate,
`validation-final-20261028170004`, whose full suite executed 28 test programs fresh, reused 4,
and passed (51/51 steps). That log is copied into this run folder as
`reused-full-from-validation-final-20261028170004.log` and named in the acceptance records.

The seal check after linking the acceptance evidence is saved as
`gate-receipt-preauthorization.log`. It is blocked only by the missing owner authorization.
`gate-receipt.log` is the gate's own earlier seal attempt, made before the evidence existed.

**Frame-time environment.** On this hybrid CPU, Windows ran the hidden test processes on
efficiency cores. The BH + 300 replay's worst frame then reached 0.45-1.8 s. That happened for
the installed 19.8.1 build too, and even with the PC idle. Pinned to the performance cores, the
same 19.9 replay ran at 97-100 ms, and 91 ms in the gate. Performance figures in this record are
render-only scenes and the cluster replay, not full gameplay frame rates.

Runs that did not count:
- `validation-final-20261028114010`: one old ejecta test failed, and one suite timed out
  because I was compiling alongside it.
- `validation-final-20261028121642`: four tests failed, all from 19.9 changes, and all fixed.
- `validation-final-20261028150248`: everything passed, but the owner's review found that
  venting still needed a free main-body slot. That is now fixed, and a new test covers it.
- `validation-final-20261028170004`: only the cluster replay failed (408 ms), from the
  efficiency-core scheduling above.

Recovery point: `checkpoints/19.9.0-before-20261028114009` (installed 19.8.1 binaries and
pre-port sources).

## Installation

Installed 2026-10-28 22:24 CDT after the owner's line `ALLOW 19.9.0
validation-final-20261028214627`. The receipt was sealed and verified before the copy, the
installed integrity check returned 0, and the receipt verified again after installing
(`receipt-postinstall.log`). Installed `ProjectKujira.exe` SHA-256:
`4CF0FE41A9F2EA4495E656CC79D0CC5BF9868FDC537B8F0409BA31F1F952CC54`. Rollback to 19.8.1:
`checkpoints/19.9.0-before-20261028114009/installed-rollback/`. Player saves were not opened.
