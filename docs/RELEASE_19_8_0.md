> Public documentation snapshot, October 28, 2026. Engine source and local gate/rollback files are not published. Results below are recorded development evidence, not new tests run for this upload.

> Historical 19.8.0 record: any candidate/pending wording below describes that earlier stage. The current release is installed 19.8.1; see [current status](DEVELOPMENT_STATUS_V19.md).

# Kujira 19.8.0 (candidate): M4 combined long-time validation and performance report

Prepared 2026-10-27 in the working copy `C:\Users\adver\kujira-dev20`, which started from the
installed 19.6.5 source (`M3_HANDOFF_2026-10-27_19.6.5.md`). Not gated or installed until the
owner approves the gate run. Settled architecture is unchanged: shared 1,024 main-body slots,
512 gas/material packets, 64 bins x 4 populations, atmosphere and charge allocations as recorded
in the handoff. No retired allocator or packet-solid behaviour was restored. Save format 37
unchanged.

## M3 review findings

Reviewed against the handoff's list (units, derivatives, contacts, recoil, entry heating,
thermal clocks, the 19.6.5 field, electricity, the magnetosphere convention, Stellar Time).

| Area | Finding | Disposition |
| --- | --- | --- |
| Softened point gravity | da_r/dr = GM(2r^2 - eps^2)/(r^2 + eps^2)^(5/2) agrees with a = -GM r/(r^2 + eps^2)^(3/2) | Correct, unchanged |
| Drag energy | dK = 1/2 mu u^2 l(2 - l) equals 1/2 mu u^2 (1 - (1 - l)^2) | Correct |
| Recoil remainder | K = mv^2/2 + v.r + r^2/(2m) from p = mv + r | Correct |
| Pair relaxation | Q = dT C_aC_b/(C_a + C_b)(1 - e^(-dt/tau)) | Correct (implemented convention) |
| Magnetopause | R_mp = R (2 B_eq^2/(mu0 P))^(1/6) is Chapman-Ferraro with the factor-2 field compression at the boundary; the generic B^2/(2 mu0 P) form omits that compression (the factor-four difference) | Correct as implemented; no change |
| 19.6.5 depth | d/R = 1 - (1 - f)^(1/3) is the shell of volume fraction f; f <= 0.8 bounds d/R <= 0.4152 | Correct |
| Breakdown | 120 Td x N gives 3.0 MV/m at 1 bar, 293 K | Correct |
| **Paid charge separation** | dq = 2W/(A + sqrt(A^2 + 4BW)) loses digits when A < 0 (other regions already favour the move) | **Fixed**: cancellation-free root for either sign (`electric.zig`), test added |
| REBOUND adapter | Force path, endpoint checks and stale-owner rejection are sound; the contact guard was an all-pairs scan on every IAS15 stage (1.28 M pairs at 1,600 particles) | **Improved**: x-sweep with a provably conservative widening (radius + 0.5 a_max dt^2); results identical to the all-pairs scan (200 random crowded stages) |

No other confirmed defect was found in the reviewed paths. Documented approximations listed in
the handoff stand.

## Performance (all bit-identical to the sequential code)

A deterministic fork-join helper (`src/parallel.zig`) runs pure per-item work on up to eight
worker threads; every order-dependent floating-point sum stays sequential, so results do not
depend on the thread count (checked by bit comparison on the full-pool scene, live radiation and
long-time thermal owners).

| Measurement (ReleaseSafe, this machine) | 19.6.5 | 19.8.0 candidate |
| --- | ---: | ---: |
| Million-year populated full-pool Stellar Time (wall) | 15.5 s | **2.2 s** |
| of which bulk thermal owners | 14.7 s | 1.6 s |
| Full-pool live physics tick (1,024 bodies, 512 packets, 64 bins), mean | 123 ms | **98 ms** |
| Radiation pass inside that tick | 38 ms | 15 ms |
| Hot full-pool tick (every solid at 2,200 K, molten), mean | not measured | 1,539 ms |

Remaining bottleneck (owner decision, not changed): when many bodies melt or evaporate in the
same tick, each phase change recomputes the body's binding energy against every other body with
the full pair-gravity kernel (O(N^2) per tick, about 0.3 us per pair, 1.5 s with 1,024 molten
bodies). Options: a persistent worker pool for these per-body sums (exact), or an approximate
binding update for distant pairs (new approximation, needs approval).

## 19.7: combined long-duration validation

`stellar_release_test.zig`: "v19.7 combined million years with an impact, a phase change and an
escape over full pools". Full shared pools, one million years and 2.5 s, with a molten 64-km body
(2,500 K), an 8-km body closing on a 16-km body at 1 km/s, and a 325-km body leaving at twice the
star's escape speed. Result: complete in 3.4 s wall (budget unchanged at 30 s); 1 impact resolved
by the existing handler (merger), 7 events, 0 unresolved; material mass exact (1e-12); packets and
bins all kept (no pool-induced loss); unique IDs; the molten body frozen by radiation; the
escaper unbound and over 100 times farther; the light ledger closed (1e-9).

## 19.8: consolidated measurements

`stellar_release_test.zig` "v19.8 performance report" prints, on every gate run:

| Item | Value |
| --- | ---: |
| Simulation (one heap object) | 38,340,608 bytes |
| Entity | 3,640 bytes |
| Full-pool save | 5,243,179 bytes |
| Full-pool tick, 1 / 8 workers, mean | 123.4 / 97.9 ms |
| Hot full-pool tick, mean | 1,539 ms |

GPU frame times stay with the existing full-pool acceptance in the rendering suite (250 ms p95
limit unchanged). Research matrix: the V19 plan section 10 sources remain the basis; no new
mechanism or calibration was introduced in M4.

## Validation

Gate `output/19.8.0/validation-final-20261027190201`: every step passed (focused 19.5-19.7 steps,
launcher guard, full Debug 51/51, rendering 1237/1238 with 1 skipped, both builds, populated
completion 8/8 run fresh, workspace integrity 3 as expected). Its full and rendering rows reused
Zig's build cache for identical test binaries; the fresh logs (`validation-final-20261027163802`
full, `validation-final-20261027183300` rendering) are copied into the run folder as `reused-*`
evidence and named in the acceptance files. Two earlier runs did not count: the first stopped
after the guard self-check missed the new 19.7 steps, and the second printed a cached completion
log. Fixes: the self-check fixture lists the 19.7 steps, and the completion run always executes
(`has_side_effects`).

Under gate load the 19.8 report measured 253 / 204 ms (1 / 8 workers) and 2.9 s hot, about twice
the idle-machine figures above; the parallel gain holds. Install awaits the owner's ALLOW line.
