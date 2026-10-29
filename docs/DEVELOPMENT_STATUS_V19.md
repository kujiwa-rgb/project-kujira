# Project Kujira: current development stage

As of **October 29, 2026**. Public documentation update only.

## Installed release

- **19.9.1** is installed; v19 milestones **M1-M4** and the 19.9 corrective releases are complete.
- Save format **37**; v19 history covers formats 26-37.
- Installed executable SHA-256: `C0514F98E72FA8126D4D40016F29AD48CEB4591A50BA13AC5456A2650C83CB69`.
- Final gate: `output/19.9.1/validation-final-20261029164106/`.
- Installed October 29 at **17:35 CDT**. The receipt was sealed and verified before copying, then verified after installation. Installed integrity returned **0**; executable, SDL2 and symbols match the receipt.
- Local rollback to **19.9.0**: `checkpoints/19.9.1-before-20261029135225/installed-rollback/`. Rollback files are not published.

These are recorded development results. No engine test, build or installation was performed for this documentation update.

## What changed in the closeout

**19.9.0:** directional debris placement at the contact seam; thermal planetary vents in gas packets rather than star-like gas bodies; unified vapour/cloud gas-layer rendering; pale-blue vapour and ice. The final candidate includes the full-table vent regression: packet capacity, not main-body space, governs ordinary vent admission.

**19.9.1:** unequal fragment sizes from a power-law tail, mass-budgeted logarithmic bins, a bounded entity budget and impact-derived size-dependent speeds. Eligible recoil chips can escape when the launch is paid; they no longer receive the old four-radius limit. Belt insertion windows and other retained legacy rules remain explicit.

These corrections retain the v19 foundation: persistent surfaces and history, Tab locking and triangle-centered steering, hard solid contacts, thermal and atmospheric accounting, local heat presentation, charge and gas-only lightning, finite-source lighting, and resumable coarse Stellar Time with analytic and linked REBOUND motion.

## Capacity and minimum-size rules

- **1,024 shared main-body slots** include all independently positioned solids.
- **512 gas/material packets** and **64 bins with up to four gas/material modes** remain separate. The old 256-solid allocator is retired.
- New physical solids remain at least **1 km in diameter**, funded from actual volume. The curtain sampler uses a **2-km draw floor** as a contraction margin, not a new physical minimum. Undrawn material stays with its donor.
- Existing sub-1-km cleanup can remove owned material and energy. This approved gameplay exception is not conservative vaporization; hot small fragments make it more visible.

## Gate evidence and conditions

- Full Debug: **51/51 steps succeeded**. Twenty test programs reused cached passes from byte-identical frozen source.
- The preceding full suite recorded **1,353 passing tests** and a runner timeout in the coherent/hard-solids program. That failed run is not represented as a passing suite. The same program executed fresh in the final gate and passed **33/33**.
- Focused 19.9 physics: **17/17**. Rendering: **1,241/1,243 passed, 2 skipped**.
- Fresh linked-REBOUND completion: **9/9**. Full-pool million years plus 2.5 seconds: **2.2797 s**; combined million-year case: **3.3997 s**, with exact requested endpoints.
- Debug and ReleaseSafe build commands succeeded using unchanged cached compile artifacts.
- The recorded gate ran at affinity **FFFF**, the 16 performance-core threads of the **i7-14700F**, without competing development work.

Copied reuse evidence remains labelled and linked to the exact candidate. Skips are not passes, and cached execution is not described as fresh.

## Performance is scene-specific

- BH+300 replay worst play frame: **81.6 ms**, under its **250-ms** limit; the recorded 19.9.0 comparison was 91.1 ms.
- Full-pool **render-only**, with physics paused: approximately **154.9-166.0 ms p95**, under its **250-ms** stress limit.
- Active `planet-gameplay-256`: **371.3 ms p95**, against its separate **375-ms** limit.
- Molten full-pool mean physics tick: **1,510.96 ms** with one worker versus **327.01 ms** with eight, about **4.62x** for that scene.

These measurements are not interchangeable and do not promise universal 60-FPS gameplay. Quiet analytic long-time windows, simplified atmospheres, entry heating and subsurface depth presentation retain their documented approximation limits. Specific prior comparison exceptions do not relax general conservation requirements.

## References

- [Updated 40-page illustrated engine reference](../UNIVERSAL_TOPOLOGY_V5_v19.pdf)
- [Complete v19 release](RELEASE_V19_COMPLETE.md)
- [19.9.1 debris and installed closeout](RELEASE_19_9_1.md)
- [19.9.0 corrective release](RELEASE_19_9_0.md)
- [19.8.1 exact-threading history](RELEASE_19_8_1.md)

## Publication scope

This repository remains **documentation-only**. Engine/game source, executables, installers, player saves, compiler caches, temporary test worlds, raw gate bundles and rollback checkpoints remain private. References to private source paths and local gate records do not publish their contents.

The existing [licensing and publication scope](../LICENSING.md) remains in force. No license change is made.
