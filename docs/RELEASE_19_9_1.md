> Public documentation snapshot, October 29, 2026. Engine source and local gate/rollback files remain private. Results below are recorded development evidence, not tests executed for this publication.

# Kujira 19.9.1: physical impact debris

Prepared 2026-10-29 in `the private development checkout` from installed 19.9.0, from the owner's
report that impact debris looked contrived: an even number of equal pieces at set masses and
speeds, in fixed directions, never escaping. The owner asked for a fragment size power law,
logarithmic size bins with mass conservation, an entity budget that protects the frame rate
and the 1,024-body ceiling, physical ejecta speeds, and escape for any surviving piece the
impact paid to launch faster than escape.

Save format 37 is unchanged. Gravity, contact detection, thermal laws, the packet pool, the
1 km minimum for new solids and every conservation ledger are unchanged.

## What 19.9.0 did

In a 4,500-km equal impact near a 0.29 Msun star, 19.9.0 made 12 debris bodies:

- 4 recoil chips, every one exactly 371 km and 2.17e20 kg (the planner's floor size);
- up to 8 curtain pieces per donor, all the same representative size;
- directions from fixed, alternating limbs;
- every speed held under 0.98 of the pair's escape speed (the v16.4 ceiling), so nothing could
  ever leave, however violent the impact.

## Changes

All of these live in `src/physics.zig` (`ejectSubFloor`, `emitDebrisCurtain`,
`emitDebrisPacket`, `trySheddingPlan`, `emitPlanetaryRingFrom`) plus one parameter in
`src/planetary.zig` (`detachHeat`).

### 1. Sizes follow the fragment tail

The ejected mass follows N(>d) ~ d^-2.85 (Leinhardt & Stewart 2012), the tail the packet model
already used (`packet_model.tail_beta`).

- **Curtain** (`emitDebrisCurtain`): the range from the largest fragment down to the smallest
  size the encounter can afford to break off (never under 2 km, see below) is split into
  factor-two (logarithmic) diameter bins. Each bin holds its share of the mass
  (`massFractionAbove`). The few bodies the entity budget allows are a mass-weighted sample of
  that tail (`massTailDiameter`, the inverse of the mass fraction). With beta = 2.85 the mass is
  spread almost evenly over the bins, so a sample spans every size, from a few large, slow
  pieces to small, fast ones. Each piece spends its own bin's mass; a bin without room for it
  draws nothing, so no bin gives out more mass than it holds.
- **2 km draw floor** (`curtain_minimum_m`): a freshly broken solid compacts under its own bulk
  solve, measured at 13 to 24 % in diameter within a few ticks. A solid under the 1 km cutoff
  leaves the world (`cleanupUndersizedSolids`, owner-approved). A first version drew pieces
  down to 1 km; pieces born at 1.0 to 1.3 km then compacted under the cutoff and left, and the
  full suite caught the missing mass (5e-12 of the total in "v16.2 comparable planets part,
  orbit or collide by gravity alone"). Drawing from 2 km keeps every new piece above 1 km after
  compaction. The 1 to 2 km share of the mass stays in the donor.
- **Recoil chips** (`trySheddingPlan`, solid recoil with two or more chips per donor): the
  planner still chooses how much mass the chips carry, with the same budgets. That mass is
  now split by diameters drawn from the same tail, between a quarter of the planner's chip
  size (two factor-two bins below it) and the size of all the chips together, then
  normalized. So one chip is usually the largest fragment and the others smaller, instead of
  n equal floor-size pieces. With four chips of 371 km, the possible range is about 90 to
  590 km. Heat, the phase ledger and atmosphere energy split by each chip's mass
  share (`detachHeat` takes the share; exactly 1.0 for equal pieces, so other paths are
  bit-identical).
- Mass that is not drawn stays in the donor. Nothing is enlarged or rounded up, and no new
  solid is under 1 km.

### 2. Entity budget (frame rate and the 1,024 ceiling)

`debrisPieceBudget` sets how many new curtain bodies one donor may create in one event, from
the free shared body slots:

| Free main-body slots | Curtain pieces per donor event |
|---|---|
| 512 or more | 8 |
| 256 to 511 | 6 |
| 128 to 255 | 4 |
| 32 to 127 | 2 |
| 1 to 31 | 1 |
| 0 | 0 |

Recoil chips keep their existing limit (at most 4, and never more than the free slots). A
crowded scene therefore gets fewer, larger-scale pieces, never more bodies than it can hold.

### 3. Speeds from the impact's energy

- One speed law per impact: v(d) = v_ref (d / d_max)^-1/3 x scatter, with a seeded scatter of
  0.8 to 1.25. Smaller pieces leave faster.
- v_ref comes from the encounter's kinetic launch share (0.20 of its heat allowance, the share
  the recoil chips already used). It is spread over the whole ejecta tail the encounter can pay
  to make (the stored rocks, plus the sub-floor share halved until affordable, down to its
  smallest affordable size) by a closed-form integral (`ejectaSpeedWeight`), not over the
  pieces that happened to be drawn. So the entity budget never changes how fast a piece
  leaves. v_ref is at most half the collision speed, and no piece leaves faster than the
  collision itself, sqrt(2 Q / reduced mass).
- A first version gave the whole launch share to the drawn pieces only. With fewer pieces
  each one got faster, so a crowded table made debris faster. That was replaced.
- Recoil chips follow the same d^-1/3 law at the same total kinetic energy as before.

### 4. Directions

- Curtain pieces: each takes a seeded side of the seam and a seeded spread about that limb
  (about 20 degrees, one sigma). The placement search stays within 40 degrees of that
  direction. Nothing wraps around the body, and there are no mirrored pairs.
- Recoil chips: a seeded limb and a seeded spread of up to 30 degrees.
- Ring ejecta: a seeded side and spread at the seam limbs.

### 5. Escape only when paid (owner decision, 2026-10-29)

The owner chose: "a piece escapes only when the impact really paid for that speed."

- The 0.98 escape ceiling on curtain pieces is removed.
- The v16.4 preflight that rejected any unbound recoil chip is removed (see recoil chips
  below for the zone check).
- Every launch is still paid in full before it happens: lift, release work, launch kinetic
  energy and fragmentation work come out of the encounter's heat, within its work allowance.
  A launch the encounter cannot pay is refused and its mass stays in the donor.

Recoil chips (owner decision, 2026-10-29, after the first gate had started): "Whether debris
can escape should follow the encounter's funded motion, not whether it came from the chip or
curtain emitter."

- The v16.2 ceiling that held a solid recoil's chips under 4 planet radii is removed for
  those chips. When the heavier body is a planet with a meaningful zone (at least 3 envelope
  radii) and below belt mass, each chip still leaves radially from the impact's centre of
  mass, now at exactly the speed its encounter paid. There is no extra kick and no guaranteed
  escape. Gravity then decides fallback, orbit or escape.
- For those chips, the preflight no longer refuses an event because a chip's orbit would
  reach beyond the zone. Leaving the planet is not leaving the stellar system: a piece that
  escapes the planet still orbits the star under the same N-body gravity.
- Unchanged: belt insertion orbits around belt-mass hosts keep their speed window and in-zone
  check, and other debris keeps the v16.2 ceiling. Detachment, launch, recoil, spin and
  gravitational costs are paid through the existing shedding ledger, and the chip size and
  speed law, the 1 km minimum and the shared limits are unchanged. Unfunded or unplaceable
  material stays with its donor.
- The first gate, `validation-final-20261029140812`, was stopped by me for this change and
  does not count.

## Evidence

Probe: iron bodies at 60 K, 2e7 km from a 0.29 Msun star, final 19.9.1 code (with paid chip
escape). The first tick with debris, then 3,400 ticks (9.4 hours) later. "Planet escape" is
speed relative to the merged planet above its escape speed; "star escape" is speed relative to
the star above the star's escape speed at that distance (the planet ignored).

| Impact | Pieces | Diameters | Speeds | Planet escape at birth | Star escape | In flight after 3,400 ticks |
|---|---|---|---|---|---|---|
| 4,500 + 4,500 km at 5 km/s | 12 | 3-387 km | 2.1-8.5 km/s | 4 (3-28 km) | 0 | 3, all leaving the planet |
| 6,000 + 6,000 km at 5 km/s | 12 | 2-446 km | 2.0-9.5 km/s | 1 (2 km) | 0 | 0 |
| 1,500 into 6,000 km at 15 km/s | 12 | 2-403 km | 0.8-9.1 km/s | 4 (the 4 chips, 332-403 km) | 0 | 10, all leaving the planet |
| 1,500 into 6,000 km at 30 km/s | 11 | 2-456 km | 1.7-19.4 km/s | 3 (the 3 chips, 368-456 km) | 2 | 3 chips, 2 leaving the star |

- Each event: up to 4 recoil chips (235-456 km, now unequal) and 8 curtain pieces spread
  across the tail.
- In the equal 5 km/s impacts the chips are below the old ceiling anyway, so they are
  unchanged and fall back. Only small curtain pieces reach planet escape there.
- At 15 km/s the 4 chips leave at 6.9-9.1 km/s against a 5.4-6.0 km/s escape: they leave the
  planet but stay bound to the star. Under v16.2 they were held to 4.2-5.1 km/s and fell back.
  Pieces from their later collisions with each other are the extra small pieces in flight.
- At 30 km/s the chips leave at 16.9-19.4 km/s, and 2 of them also leave the stellar system
  (the star's escape speed there is about 62 km/s against a 44 km/s orbit).
- In the curtain the largest pieces are the slowest and the smallest the fastest, as the speed
  law sets. Curtain directions stayed within about 40 degrees of the seam limbs (0 and 180
  degrees in these scenes).
- A first version gave the curtain's launch share to the drawn pieces only, and 8 of 12 then
  escaped at 10-13.6 km/s in the 15 km/s hit. That speed depended on how many pieces the budget
  allowed, and was replaced (section 3).

Long runs, 20,000 ticks each:

| Impact | Pieces seen | Smallest diameter | Left under the 1 km cutoff |
|---|---|---|---|
| 4,500 + 4,500 km at 5 km/s (chips capped, before the chip change) | 43 | 1.9 km | 0 |
| 1,500 into 6,000 km at 30 km/s | 66 | 2.0 km | 0 |
| 1,500 into 6,000 km at 15 km/s | 38 | 1.0 km | 2 |

In the 15 km/s run, two 2 km curtain pieces at about 20,000 K evaporated their solid into their
own vapour (`applyMaterialPhase`), so their solid diameter shrank to 1 km within about 500 ticks.
The existing owner-approved cutoff (`cleanupUndersizedSolids`) then removed them with their
inventory: 4.5e13 and 6.0e13 kg, about 1e-10 of the planet. This is the engine's existing rule
for any hot solid under 1 km, not a 19.9.1 path, but 19.9.1 makes small hot pieces more common.
The material mass of the 4,500 km scene changed by 3e-14 of the total.

## Tests

- New: "v19.9.1 debris tail: sizes, speed law and entity budget follow their formulas". It
  checks the count and mass inverses of the tail (the mass draw against `massFractionAbove`,
  1e-9), the closed-form speed weight against a numeric sum (1e-6), the kinetic budget and
  collision-speed cap, and the budget table.
- "v19.9 equal impact ejecta leave as two opposite bounded jets from the contact seam" now also
  requires every piece to be at least 1 km and the pieces to differ in mass.
- Changed to the owner's rule (superseded contracts):
  - "v16.4 planet-collision debris near a star stays below the system's escape speed" is now
    "v19.9.1 planet-collision debris near a star escapes only on paid launch work". A piece
    above escape requires paid launch work, and total mass must match to 1e-12.
  - "solid recoil paid impact leaves outward chips outside the combined remnant in both solvers"
    required every chip to be exactly the floor size, bound, and with its apoapsis inside the
    zone. It now requires each chip to keep its parent's density, be at least 1 km, and the
    chips together to carry the plan's mass (1e-12). A chip may leave the zone or escape only
    when its launch work was paid. Its placement, periapsis, count and budget checks are
    unchanged.
  - The landing test "v19.5 event-local energy: complete radial orbital and repeated
    shared-body landings" counts fragments still in flight, because a power-law curtain can
    excavate more than the impactor brings.
  - Three hard-solid curtain tests pass the event speed law.

## Validation

Before the gate, in `kujira-dev20` (ReleaseSafe): the full physics suite passed 1,074 of
1,074, and the zone, debris and recoil set passed 225 of 225.

Final gate `output/19.9.1/validation-final-20261029164106`. Every step passed. It ran with
processor affinity FFFF (the 16 performance-core threads of the i7-14700F), with nothing else
running.

| Step | Result | Fresh or cached |
|---|---|---|
| Focused steps (all, including `focused-19.9-physics`, 17 of 17) | 0 | ran |
| Launcher guard | 0 | ran |
| Full Debug | 51/51 steps | 20 test programs reused cached passes; the coherent/hard-solids program ran (33/33) |
| Rendering (ReleaseSafe) | 1,241/1,243 passed, 2 skipped | ran |
| BH + 300 cluster replay | worst play frame 81.6 ms (limit 250; 19.9.0: 91.1 ms) | ran |
| Populated completion | 9/9, linked REBOUND | ran (it always runs fresh) |
| Debug and ReleaseSafe builds | 0 | ran |
| Workspace integrity | 3, as expected | ran |

The full-suite cache reuse is genuine. The frozen source (206 files) is byte-identical to
`validation-final-20261029150554`, whose full suite executed 16 test programs fresh, reused 4
and passed 1,353 tests. Its one failed program, coherent/hard solids, stopped on a "test
runner failed to respond" timeout. The same binary (identical hash) passes 33 of 33 in 2
seconds through the build system, and it executed fresh in the final gate and passed. That log
is copied into the final run folder as `reused-full-from-validation-final-20261029150554.log`
and named in the acceptance records.

The seal check after linking the acceptance evidence is saved as
`gate-receipt-preauthorization.log`. It is blocked only by the missing owner authorization.

Runs that did not count:
- `validation-final-20261029140812`: stopped by me when the owner chose paid escape for
  recoil chips (that candidate could not qualify the new one).
- `validation-final-20261029150554`: everything passed except the runner timeout above. The
  gate script then kept running its rendering step after the full-suite failure, and that
  step overlapped my rerun of the coherent program and the start of the final gate. It failed
  one live-acceptance test there. The final gate ran rendering with the machine to itself and
  passed it, with frame figures matching 19.9.0 (planet-gameplay-256 p95 371 ms against
  375 ms).

Recovery point: `checkpoints/19.9.1-before-20261029135225` (installed 19.9.0 binaries, exe
SHA-256 `4CF0FE41A9F2EA4495E656CC79D0CC5BF9868FDC537B8F0409BA31F1F952CC54`, and the pre-port
sources).

## Installation

Installed 2026-10-29 17:35 CDT after the owner's confirmation of `ALLOW 19.9.1
validation-final-20261029164106` (an external review of the evidence found no blocker). The
receipt was sealed and verified before the copy (`gate-receipt-final.log`, `install-final.log`),
the installed integrity check returned 0 (`integrity-installed-final.log`), and the receipt
verified again after installing (`receipt-postinstall.log`). Installed `ProjectKujira.exe`
SHA-256: `C0514F98E72FA8126D4D40016F29AD48CEB4591A50BA13AC5456A2650C83CB69`, byte-identical to
the gated build, as are `SDL2.dll` and `ProjectKujira.pdb`. Rollback to 19.9.0:
`checkpoints/19.9.1-before-20261029135225/installed-rollback/`. Player saves were not opened.
