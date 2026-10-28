> Public documentation snapshot, October 28, 2026. Engine source and local gate/rollback files are not published. Results below are recorded development evidence, not new tests run for this upload.

# Kujira 19.8.1 (candidate): exact threading of molten-heavy ticks

Prepared 2026-10-28 in `C:\Users\adver\kujira-dev20` from installed 19.8.0. The owner chose exact
threading over an approximation. These stay the same: equations, phase rules, accumulation
order, shared-transfer order, saves (format 37), controls, capacities and conservation
tolerances. No fast-math, reassociation or new approximation.

## What was slow

Profile of the hot full-pool scene (every solid heated to 2,200 K, 1,024 bodies, 512 packets,
64 bins; ReleaseSafe, 8 workers, after the first change):

| Stage | Cost | Cause |
| --- | ---: | --- |
| Radiation | 700 ms | Every glowing body lights every other: two O(N^2) passes over field beams (`absorbFieldLightMode`) |
| Thermal | 280-350 ms | Each melting or relaxing body re-sums its potential against all bodies (`pairPotential`), plus all packets and bins (`packetPotentialOne`) |

## Changes (all exact)

- `src/parallel.zig`: the fork-join helper now uses a small persistent pool. Helpers spin briefly
  and then sleep on the job counter (`RtlWaitOnAddress`), so thousands of short jobs per tick cost
  microseconds each instead of a thread spawn. A nested or concurrent request runs inline. The
  chunks are unchanged.
- `addPairPotentialChange`: one helper replaces six copies of the "body changed, re-book the pair
  potential" loop (`applyMaterialPhase`, `relaxPlanetaryEnvelope`, `applyCrustChemistry`,
  `commitChemistry`, `bulkBodyRadius`, `contractBody`). `bookGravityChange` and envelope shedding
  keep their own loops (different skip rules). Each term is the same expression. The terms are computed in parallel
  and added to the binding ledger one at a time, in body order, starting from the same value.
- `packetPotentialOne`: the same packet and bin terms, computed in parallel, summed in the
  original order (packets, then bins).
- `absorbFieldLightMode`: each source's demand sums its receivers in ascending order, and each
  receiver's intake sums its sources in ascending order, exactly as the nested loops did. Only
  the independent items run in parallel. Payment between the two passes is unchanged and
  sequential.

## Measured

Same scene, same machine, four ticks each:

| | Before (19.8.0) | After (19.8.1) |
| --- | ---: | ---: |
| Hot full-pool tick, 1 worker | 1,530 ms | 1,542 ms |
| Hot full-pool tick, 8 workers | 1,496 ms | **395 ms** |

State agreement: field-by-field dumps of the scene after four ticks (bits of mass, heat
reservoirs, binding, radius, position, velocity, atmosphere, envelope and every skin sector)
are identical: 19.8.0 at 1 and 8 workers, and 19.8.1 at 1 and 8 workers (SHA-256 `E86270AB…`
for all four). The new test "v19.8.1 molten-heavy full-pool ticks agree bit for bit at one and
eight workers" checks this on every gate run.

What remains in that tick at 8 workers: thermal about 180 ms (of which pair re-booking about
105 ms, bounded by one hand-off per changed body), radiation about 125 ms, contacts about 65 ms.

## Validation

Gate `output/19.8.1/validation-final-20261027231328` ran every step fresh and all passed: focused
19.5-19.7 steps, launcher guard, full Debug 51/51 (1383/1388, 5 skipped), rendering 1238/1239
(1 skipped; full-pool live p95 179-191 ms against the unchanged 250 ms limit), both builds,
populated completion 9/9 including the molten agreement test, workspace integrity 3 as expected.
Acceptance evidence is linked in the run folder.
