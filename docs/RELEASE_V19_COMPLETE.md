> Public documentation snapshot, October 28, 2026. Engine source and local gate/rollback files are not published. Results below are recorded development evidence, not new tests run for this upload.

# Project Kujira v19: complete release

**Installed release: 19.8.1, 2026-10-28.** v19 milestones M1 to M4 (parts 19.1 to 19.8) are
gated and installed. 19.9 is reserved for fixes. Save format 37.

This document gathers every v19 change and the engine models behind it. The per-part release
notes stay authoritative for their own detail, measurements and history. Where they disagree,
the later record wins: final receipts beat stage documents, and stage documents beat plans.

| | |
|---|---|
| Project root | `GEgine/` (authoritative gated tree) |
| Installed executable | `%LOCALAPPDATA%\Programs\ProjectKujira\ProjectKujira.exe` |
| 19.8.1 SHA-256 | `24D7185D3EC67231523592826B8BB5235EA5A80C9D1730577CF6665FD3878B76` |
| Final gate | `output/19.8.1/validation-final-20261027231328/` (sealed receipt `gate-passed.json`) |
| Rollback to 19.8.0 | `checkpoints/19.8.1-before-20261027231328/installed-rollback/` |
| Toolchain | Zig 0.17.0-dev.2085, SDL2, OpenGL 3.3, REBOUND 5.2.2 (vendored, commit c3413471) |
| Player saves | `%LOCALAPPDATA%\ProjectKujira` (never opened by development or gates) |

Contents:

1. Central contract
2. Milestones and release history
3. Shared capacities and size rules
4. Engine models (units, gravity, contacts, material pools, thermal, radiation, atmosphere,
   electricity, magnetism, light and shadow, Stellar Time, surfaces and view, controls)
5. Performance architecture
6. Persistence and save formats
7. Validation and release protocol
8. Approximations, exceptions and limits
9. Record index

---

## 1. Central contract

> Material keeps its physical inventory and history at every zoom. Grouping for efficiency is
> not physical merging. Zoom and time alone cannot create or erase matter.

What v19 holds to:

- **One material ledger.** Every kilogram and joule belongs to exactly one body, packet,
  bin population or named ledger (escaped light, binding, separated charge, and so on). Every
  transfer is a named transaction with exact mass, momentum, angular momentum and energy
  accounting.
- **Observation changes nothing.** Camera, zoom, focus, hovering, sampling and rendering read
  state only. Tests compare the world field by field before and after drawing.
- **The collision solver stays authoritative.** Surface records, cloud labels and colour
  fields describe outcomes the solver already decided. They never feed back into physics.
- **Determinism.** Parallel work (19.7 onward) computes independent items in parallel and
  keeps every order-dependent sum sequential, in the original order. Results are
  bit-identical for any worker count.
- **No hidden solids.** New physical solids are at least 1 km across (section 3). Packets and
  bins hold gas and material modes, not a second population of rocks.

---

## 2. Milestones and release history

| Milestone | Parts | Theme | Final state |
|---|---|---|---|
| M1 | 19.1-19.2 | Surface view, 10-km triangle, persistent surface records | Installed (format 27) |
| M2 | 19.3-19.4 | Material packets and bins, loose ground, small-material encounters | Installed (format 29) |
| M3 | 19.5-19.6.5 | Thermal continuity, clouds and rain, charge and lightning, unified solids, corrections | Installed as 19.6.5 (format 37) |
| M4 | 19.7-19.8.1 | Combined long-time validation, performance report, exact threading | Installed as 19.8.1 (format 37) |

### Part by part

| Release | Date | What it delivered | Format |
|---|---|---|---|
| 19.1.0 | 10-20 | Read-only surface view to zoom 3,750 (100 m/px) while focused. Universal 10-km triangle with a subpixel bracket. Side-on framing. Body-fixed hovering above the drawn surface. | 26 |
| 19.2.0 | 10-21 | Surface records: 64 body-fixed columns per body with deposits, seams, damage and melt. Snapshot-then-commit merges. Donor history moves to the survivor. Fragment inheritance. Frame watchdog. | 27 |
| 19.3.0 | 10-21 | Packet pool and 64 bins of 4 populations. Body-packet gravity from detachment. Decision A: ring ejecta leave as packets when the body table is full. Decision B: sub-floor ejecta inside the existing budgets. Kepler paths in Stellar Time. Cosmetic sample pool. | 28 |
| 19.4.0 | 10-21 | 512 packets. Loose ground layer. Landing classes (terrain, ricochet, loose). Sliding above the supported slope. Cloud-cloud stick, bounce and fragment. Loading screen and crash log. Infall interception fix. | 29 |
| 19.4.1 | 10-21 | Star hand-off rim fix. Realistic preset spacing (galaxy 5.2 ly, clusters 0.05 ly). Black-hole framing and close-up lensing. | 29 |
| 19.4.2 | 10-22 | One refining raster (cells halve every factor sqrt 2 of zoom). Rim-band sector heat. Locked focus. Triangle-anchored zoom. Contact-point merger heat. | 30 |
| 19.5.0 | 10-22 | 19.5a cloud ownership. 19.5b packet and bin radiation. 19.5c resting columns and water table. 19.5d condensation, settling, rain and drag. Starlight that only heats. Owed half-kick at contacts. Glow from the emitting skin. LOCK/UNLOCK FOCUS and analog steering. | 30-31 |
| 19.6.0 | 10-22 | Gaussian charge regions. Paid mixed-phase charging. Bounded air-like breakdown. Intracloud and ground strokes. Lightning drawing. Magnetosphere compatibility. | 32 |
| 19.6.1 | 10-22 | Tab focus toggle. Current heat drawn over old deposit colour. Steering axes and sensitivity. | 32 |
| 19.6.2 | 10-22 | Bounded relief allocation. Constituent-size debris samples. Viewport-aware sampling. Packet-only ejecta (owner-approved). Deep-zoom culling. Launcher and gate guard. | 32 |
| 19.6.3 | 10-25 | Persistent IDs. Coupled motion and contact. Linked REBOUND adapter. Coarse Stellar Time scheduler. Finite-source light and shadow. 10 K stall fix. | 33-35 |
| 19.6.4 | 10-27 | Unified solids in the 1,024 main-body slots. Within-capacity legacy migration. Authoritative momentum remainder (drag recoil). Directional entry heating. Atmospheric broad phase. | 36-37 |
| 19.6.5 | 10-27 | Curved subsurface impact-heat colour field on both raster paths. M3 complete. | 37 |
| 19.8.0 | 10-27 | M3 review fix (charge-separation root). Swept REBOUND contact guard. Deterministic parallel radiation and long-time thermal owners. 19.7 combined million-year case. 19.8 performance report. | 37 |
| 19.8.1 | 10-28 | Exact threading of molten-heavy ticks: persistent worker pool, shared pair-potential helper, parallel packet potentials and field light. M4 complete. | 37 |

Plans that were not built as written:

- The 128 x 64 local tile and its refine/restrict exchanges (19.3) were replaced by the packet
  and bin pools.
- M3_PLAN.md replaced the broader electrical/magnetic scope: no electromagnetic terrain, no
  shield system, and the magnetosphere proxy is unchanged.
- Composite bodies (contact binaries) stay unsupported. Merged solids are spherical.

---

## 3. Shared capacities and size rules

| Allocation | Meaning |
|---|---|
| 1,024 main-body slots | Stars, planets, spawned bodies and every individually placed solid product share them |
| 512 material packets | Gas and material clouds; not disconnected-rock storage |
| 64 bins x 4 populations | 256 gas/material modes, each with its own composition, mass, heat, phase and charge state; never averaged |
| 1,024 cosmetic samples | Per-frame display budget (at most 64 per cloud); no physical inventory |
| 1,024 surface records | 64 columns plus 8 recent events per body |
| 256 atmosphere table owners | 64 columns x 3 layers each; one detailed owner at 64 x 12; 16 descriptors |
| 256 charge regions | Electrical state, not bodies |

Size rules:

- **1 km minimum** physical solid diameter for every new solid, including constituents
  represented by packets or bins. Size comes from retained volume and count. An undersized or
  unfunded product is not born: its material and work stay with the donor. Nothing is rounded
  up, invented or discarded to qualify.
- The 375-km chip/raster reference is a display and resolution scale. It is not the birth
  minimum or permission to enlarge a subpixel solid.
- Disconnected pieces are not one equivalent sphere: N spheres of radius r have projected
  area N pi r^2, and halving a diameter divides mass by eight.
- Owner-approved cleanup exception: an existing solid below 1 km merges if it really
  contacts another body; otherwise the isolated owner retires. This isolated case removes
  material and energy (nonconserving) and applies only there.
- A world that does not fit the shared capacities is rejected atomically on load.

---

## 4. Engine models

Notation: SI units where named (kg, m, s, J, C, Pa). `world.km_per_unit = 3000`, so
`meters_per_unit = 3e6`. Simulation seconds are not render seconds. One live tick is 10 s.

### 4.1 Volume, density and size

$$V=\sum_k\frac{m_k}{\rho_k(T,p,\text{phase})},\qquad d=2\left(\frac{3V}{4\pi}\right)^{1/3}$$

Each constituent and layer uses its own supported density. Phase or thermal change can change
condensed radius without changing retained material. Code: `material_volume.zig`,
`material_phase.zig`, `scale.zig`.

### 4.2 Gravity

Softened point source outside structured interiors:

$$\mathbf a=-GM\frac{\mathbf r}{(r^2+\epsilon^2)^{3/2}},\qquad
\Phi=-\frac{GM}{\sqrt{r^2+\epsilon^2}},\qquad
\frac{da_r}{dr}=GM\frac{2r^2-\epsilon^2}{(r^2+\epsilon^2)^{5/2}}$$

- World acceleration uses `G_world = G_SI / meters_per_unit^3`.
- Planetary pairs use a local-size softening from both gravity radii, with gas clouds softened
  as distributed mass (`imminent_accretion.cloudSoftening`). Stars and remnants keep the
  stellar kernel.
- Red giants and supergiants use a structured core plus envelope profile with matched force,
  potential and derivative (`stellar_gravity.sampleProfile`, envelope polynomial).
- Packets and bins attract and are attracted by every body, softened by their extent, from
  the moment of detachment. Packet-packet, packet-bin and bin-bin gravity are omitted (a
  recorded v19 approximation).
- **Pair-potential ledger.** Any event that changes a body's mass, radius or gravity profile
  re-books its potential against every other body and every packet and bin into the body's
  unresolved binding energy:

  $$E_{bind}\mathrel{+}=\sum_{o\neq i}\big[\Phi(\text{before},o)-\Phi(\text{after},o)\big]
  +\sum_{p}\big[\Phi(\text{before},p)-\Phi(\text{after},p)\big]$$

  19.8.1 routes six of these call sites through one exact helper, `addPairPotentialChange`
  (section 5).

### 4.3 Contacts, mergers and ejecta

- Contact comes from physical exteriors and swept paths. A glow, gas envelope or population
  extent is not a solid surface. A miss gets no impulse.
- Two-body quantities: reduced mass `mu = mM/(m+M)`, relative velocity `u = v - V`,
  `K_rel = 1/2 mu |u|^2`.
- Mergers add mass and authoritative momentum, derive size from volume, and allocate heat,
  spin and ejecta through existing budgets. Equal-mass merger heat lands at the contact point
  between the surfaces (19.4.2).
- **Owed half-kick (19.5).** Contacts resolve on mid-step drift positions, so the mutual
  gravity from contact to midpoint is applied reciprocally before the event. It is removed if
  the event does not happen. Event-local energy closes to about 2e-5 of the contact energy.
  The correction is never booked as heat.
- **Ejecta (19.3 decision B, 19.6.2).** Ejected mass is the encounter's existing mass-loss
  limit. Chips follow the existing rule. The remainder leaves as packets at the constituent
  size the work budget can pay, using `N(>D) ∝ D^-2.85` (Leinhardt & Stewart 2012) from 1 km to
  the largest fragment, at the Benz & Asphaug (1999) disruption cost. Eligible contacts may
  queue packet-only ejecta when chip resolution fails. The curtain is symmetric about the
  impact axis, so recoils cancel. Launch stays under 0.98 of escape speed including donor
  recoil. Unspent work stays with the donor.
- **Unified solids (19.6.4).** Every independent solid occupies a main-body slot. Births are
  refused before any donor mutation if unfunded, undersized, overlapping or unplaceable.

### 4.4 Material packets and bins (19.3-19.4)

- **Packet:** a dormant inventory (every species, trace, phase and heat), position and
  velocity, extent, representative constituent diameter and multiplicity, parent ID and
  birth tick, and bound/unbound state.
- **Bin:** exact mass moments (mass, centroid, bulk velocity, second moments of position and
  velocity) and up to 4 populations. Populations merge only when compatible: composition
  within 5 %, the same dominant phase and specific heat within 2 %, by exact sums. Bins take
  only material within twice the larger spread. A transfer with no conservative
  representation is refused and counted (`unresolved_capacity`).
- **Landing classes (19.4),** host-relative only:
  - welded terrain when molten, or when `v^2/2 >= c_p (T_melt - T) + L`;
  - ricochet below 15° (Gault & Wedekind 1978): up to half leaves at half the tangential speed
    as a cloud of twice the extent and half the constituent size;
  - loose otherwise.
- **Interception:** an intact rock lands whole. A distribution loses only the share whose
  paths hit (a uniform strip for packets, Gaussian for bins), using the path's own lateral
  miss.
- **Cloud-cloud encounters:** rate `N_a N_b sigma v dt overlap / area`. Stick below mutual
  escape speed, fragment at the Benz & Asphaug Q*_D, bounce between. Restitution is 0, 0.5 and
  0.1 respectively, and the kinetic energy lost becomes heat. Sticking grows constituents and
  never makes one rock.
- **Sliding:** loose ground moves only where the slope exceeds the angle of repose
  (tan 0.70 rock, 0.60 ice), lowered by impact shaking. The released potential
  `sum rho g A h^2/2` becomes heat taken from binding. Time alone never moves loose ground.

### 4.5 Thermal state and phase

- Each body has a core (0.20 of the solid mass), eight skin sectors and a retained atmosphere:

  $$U_s=\max(0,U_{total}-U_c-U_{atm}),\qquad U_{s,i}=w_iU_s,\quad \sum w_i=1$$

- Temperature comes from inverting enthalpy, with each constituent's pressure-dependent
  melting interval:

  $$h(T,p)=\sum_k x_k\,[c_{p,k}T+L_k f_{melt,k}(T,p)]$$

- Pair relaxation (implemented convention):

  $$Q=(T_a-T_b)\frac{C_aC_b}{C_a+C_b}\left(1-e^{-\Delta t/\tau}\right)$$

- Each skin sector exchanges heat with its outer gas region. Inner/outer and lateral transport
  redistribute it. Gas heat patterns advect by `0.15 ω dt`, conservatively.
- **Phase change:** `phasePlan`/`applyPhasePlan` move mass between phases. Radius follows
  `r' = r (V'/V)^{1/3}`. Spin energy released or required is paid from or returned to the
  skin. Pair and packet potentials are re-booked (4.2). Water, species and condensate ledgers
  are authoritative. Chemical and latent energy are never credited twice.
- **Clock constants**, each applied by its own call path and never multiplied blindly:
  - `thermal_clock_scale = 100`;
  - unresolved wind `wind_clock_boost = 1e5`;
  - eligible substellar gas cooling fraction 0.01, on the positive net deficit only:
    `P_net,game = max(0, P_phys - P_heat) / 0.01`.

### 4.6 Radiation, field light and glow

- Absorption and emission:

  $$P_{abs}=\alpha\frac{L}{4\pi D^2}A_{proj},\qquad P_{emit}=\varepsilon\sigma A(T^4-T_{bg}^4)$$

- **Starlight only heats (19.5).** Ignited stars and glowing hot bodies supply absorbed power
  `dU = P_abs - P_emit`, directional on a planet's skin and shadowed by hosts. The source pays
  from its core above onset (a star) or from its envelope or skin after its own balance (a hot
  body). Payment goes through a prepaid credit of at most one rounding step, released as
  escaped light if the source disappears.
- **Field-light passes:** each source's demand sums its receivers' beams. Payment is
  sequential. Each receiver's intake then sums its sources' beams scaled by the paid share.
  19.8.1 runs both passes in parallel with the same summation order (section 5).
- **Packets and bins (19.5b):** N constituents of radius r over extent e have
  `tau = N r^2 / e^2`. They absorb `pi e^2 (1 - e^{-tau})` and emit four times that. Bin
  populations shade one another. The grey equilibrium `T_* sqrt(R_*/2D)` is reproduced
  (278.33 K at 1 AU).
- **Glow** reads the emitting skin: `T_rad = (1/8 Σ T_{s,i}^4)^{1/4}`. A hot face glows without
  gas. A hot core under cool skin does not glow. There are no fade timers.
- Light ledgers (radiated, escaped, absorbed, field-absorbed, core-radiated) close to 1e-9 in
  the acceptance scenes.

### 4.7 Atmosphere, clouds, drag and entry heating (19.5, 19.6.4)

- **Columns:** at most five scale heights, `H = R_gas T / (μ g)`. Each column is exactly
  hydrostatic. Temperature follows the dry adiabat `g/c_p` to a radiative floor at 0.75 of the
  surface temperature. Gas giants use a cloud-deck reference radius.
- **Clouds** label water the saturation equilibrium already holds condensed. Formation pays
  the lift into binding. Settling and evaporation return it. Rain clears the label.
  Processes: formation and evaporation (about 1,000 s), warm-rain growth, terminal settling
  (Epstein, Stokes or Newton by Knudsen and Reynolds number) and bounded eddy mixing. All are
  exponential or implicit. Live columns update every 600 s, 8 bodies per update.
- **Drag** with stopping time t_s:

  $$\ell=1-e^{-\Delta t/t_s},\quad \Delta\mathbf p=-\mu\ell\mathbf u,\quad
  \Delta K=\tfrac12\mu|\mathbf u|^2\ell(2-\ell)$$

  The host takes the opposite impulse. The Epstein branch applies when the mean free path
  exceeds the constituent diameter.
- **Momentum remainder (19.6.4):** below-representable momentum is authoritative motion,
  `p = m v + r_p`, with `K = |p|^2/(2m)` and drift `r_p Δt / m`. It is carried through
  split/merge and saves.
- **Entry wall demand** (air-like carrier only):

  $$q=\frac{q_cq_k}{q_c+q_k},\quad q_c=1.7415\times10^{-4}\sqrt{\rho_g/r_n}\,|u|^3,\quad q_k=\tfrac12\rho_g|u|^3$$

  Demand is scaled by projected area, a wall-enthalpy weight, a Mach join `(M^2-1)/24` and the
  carrier fraction, then capped by the drag energy: `Q_s = min(ΔK, Q_demand)` and
  `Q_g = ΔK - Q_s`. Solid heat goes to the windward sectors. Entry deposits nothing into the
  core.

### 4.8 Electricity and lightning (19.6)

- **Gaussian charge regions** (256 shared), body-fixed, with energy

  $$U=\tfrac12\sum_{ij}q_iq_jK_{ij},\quad K_{ij}=k\frac{\operatorname{erf}\!\big(r/\sqrt{2(s_i^2+s_j^2)}\big)}{r},\quad K_{self}=\frac{k}{\sqrt\pi\,s}$$

  The energy is finite and never negative.
- **Moving charge** dq changes U by `ΔU = A dq + B dq^2`, with `A = V_up - V_low` and
  `B = K_uu/2 + K_ll/2 - K_ul`. Paid separation of work W solves

  $$dq=\begin{cases}\dfrac{2W}{A+\sqrt{A^2+4BW}},&A\ge0\\[2mm]\dfrac{\sqrt{A^2+4BW}-A}{2B},&A<0\end{cases}$$

  This cancellation-free form for either sign was added in 19.8.0.
- **Charging** happens only in mixed-phase layers (ice with supercooled drops), using at most
  1e-3 of the settling work. Heat or compression alone never charges.
- **Conductivity** by identity and phase: metallic Irio, Alumini and Gol. The body-level
  percolation share is 0.31. Conductors store and spread charge; they never charge on their
  own.
- **Breakdown** applies only to air-like N2/O2 gas: about 120 Td (3 MV/m at 1 bar, 293 K;
  Raizer 1991). The bounds are 180-1,500 K, 0.01-10 bar and gaps of 1 m to 1,000 km. Cloud
  enhancement uses 0.1 of the uniform threshold, and charge flows until the field falls to 0.3
  of it. Unsourced gases never discharge.
- **Discharge** energy `U_before - U_after` is 1 % light and the rest gas heat. Ledgers close
  to 1e-6, and the universe total to 1e-12.
- **Drawing:** a white/blue-white core, blue glow and violet fringe, driven by released energy,
  pressure and display age (0.35 s cosmetic fade). Stored charge draws nothing.

### 4.9 Magnetosphere (inherited proxy, unchanged)

$$B_{eq}=\frac{\mu_0 m_{dip}}{4\pi R^3},\qquad R_{mp}=R\max\!\left(1,\left[\frac{2B_{eq}^2}{\mu_0P_{wind}}\right]^{1/6}\right)$$

This is the equatorial Chapman-Ferraro standoff with the factor-2 boundary compression; the
19.8.0 review confirmed the convention. The dipole moment comes from conducting interior
material, temperature and mass eligibility, spin and remanence. It is not an MHD dynamo.
Coupling uses ionization and plasma proxies, never stored charge. Magnetic results are
byte-identical with and without separated charge.

### 4.10 Finite-source light and shadow (19.6.3)

For source s, blocker b and point p: `t = ((b-s)·(p-s))/|p-s|^2`, `w = R_*(1-t)`,
`m = |b - s - t(p-s)|`. Then `coverage = min(1, R_b^2/w^2) S(e)` and
`T = 1 - opacity · coverage`, with `S(x) = x^2(3-2x)`. This is a bounded beam-coverage
approximation, not exact disk overlap or limb darkening. Display lighting uses at most eight
casters.

### 4.11 Stellar Time (19.3 paths, 19.6.3 coarse scheduler)

- **Coarse mode** (automatic for requests of 100 years or more) advances a future state:
  - qualified analytic orbits or linked REBOUND IAS15 integration;
  - bulk heat and material evolution;
  - stellar aging;
  - consequential supported encounters.

  It then continues to the exact requested endpoint. It does not replay weather, lightning or
  10-second frames.
- **Packets and bins** follow Kepler paths about their local primary. The primary takes the
  reciprocal two-body correction.
  - Below 64 orbits per step, encounters use swept segments.
  - Above 64 orbits, an Öpik/Wetherill orbit-averaged rate `2 R v_rel / (π(Q^2-q^2))` applies.
- **Quiet windows** grow toward the destination and stop at the earliest predicted contact.
  Budget yield is resumable work.
- **REBOUND adapter:**
  - the force callback projects trial positions and calls Kujira's own acceleration;
  - a guard callback runs on every force evaluation;
  - stale owners are rejected.

  19.8.0 replaced the all-pairs contact guard (1.28 M pairs at 1,600 particles) with a
  conservative x-sweep. Each interval is widened by `radius + ½ a_max dt^2`. Results are
  identical to the all-pairs scan, which remains as the reference and fallback.
- **Bulk thermal owners** use the same laws. Quietness bounds change over the remaining
  passive interval and is invalidated by renewed light, impact or phase change.

### 4.12 Surfaces, raster and view (19.1-19.6.5)

- **Surface records:** 64 body-fixed columns plus 8 recent events. Deposits are
  volume-neutral (`Σ w_i r_i^3` constant), with the footprint widened until the peak fits
  `min(2 % R, 1e8 Pa / (ρ g_eff))`. Seams scale as 1 - melt and damage as
  `1 - exp(-E_heat/(m·1e4 J/kg))`. The key is ID plus birth tick. Merges snapshot first and
  commit after the solver.
- **Relief:** a seeded 24-harmonic visual profile capped at 2 % of the radius and flattened by
  melt. It uses a 1,024-sample lookup.
- **Water level** solves `Σ w_i (L^3 - r_i^3)_+ = R_w^3 - R_c^3`.
- **One refining raster:** above zoom 48 the cell is 375 km / 2^n with
  `n = floor(2 log2(zoom/48))`, down to 1.46 km. Earlier edges stay put. Zoom never allocates
  material.
- **Eight-sector heat** is drawn as a rim band. 19.6.5 adds a curved subsurface field:
  - `f_i = c e_i/(e_i + h_0)` and `d_i/R = 1 - (1 - f_i)^{1/3}`, bounded by 0.4152;
  - normalized angular weights `max(0, n_i·q̂)^4`;
  - a depth mask `1 - S(δ / (d/R))` with smoothstep S.

  It is a presentation of existing heat. It does not change the silhouette or the physics.
- **Triangle and bracket:** the triangle is 10 km nose to base (side 11.547 km). Bracket
  ticks show while the triangle is under 0.75 px tall and hide at 1.0 px or more. Clearance
  covers the full footprint over drawn cells. Gas giants use their envelope as the floor.
  Navigation is massless.

### 4.13 Controls (19.5-19.6.1)

- **Focus:**
  - Selecting a body locks focus.
  - LOCK FOCUS / UNLOCK FOCUS button, and Tab as its keyboard twin (no key repeat).
  - Pointer movement never releases focus.
- **Steering:** the triangle is the neutral point. All values are at a 600-px short side and
  scale by `max(0.25, short/600)`:

  | Setting | Value |
  |---|---|
  | Dead zone | 24 px |
  | Full deflection | 260 px |
  | Tangential cap | 360 px/s |
  | Altitude cap | 180 px/s |
  | Response | 0.12 s, exponential lag |
  | Maximum altitude | 35 % of the short side |

  Wheel zoom anchors on the triangle.

---

## 5. Performance architecture (M4)

### 5.1 Deterministic parallel helper (`src/parallel.zig`)

- `forRange(count, min_per_worker, ctx, func)` splits [0, count) into contiguous chunks. The
  caller runs chunk 0; pool helpers run the rest. Up to 8 workers (`max_workers`), capped at
  CPU count - 1.
- **Persistent pool (19.8.1):**
  - helpers start on first use, spin about 20,000 pause cycles, then sleep on the job counter
    (`RtlWaitOnAddress`);
  - every helper acknowledges every job, so none can read a job while the next is written;
  - a nested or concurrent request runs inline.

  Thousands of short jobs per tick cost microseconds each.
- **Rule:** jobs write only their own items. Every order-dependent floating-point sum stays
  sequential in the original order. There is no fast-math or reassociation.

### 5.2 Parallel passes (all exact)

| Pass | Since | What runs in parallel | What stays sequential |
|---|---|---|---|
| Radiation rows | 19.8.0 | Each source's per-receiver weights (`radiationRow`) | Ledgers; receivers sum sources in order |
| Planet emission and balance | 19.8.0 | `cooling(...)` and `coolingDirectional` per planet | Commit loop |
| Long-time thermal owners | 19.8.0 | `solveBulkBodyMode` estimates and balances | Radius and binding settlement; errors rethrown in order |
| Pair-potential re-booking | 19.8.1 | Per-pair terms `Φ(before,o) - Φ(after,o)` | Added to binding one at a time in body order |
| Packet potential of one body | 19.8.1 | Per-packet and per-bin terms | Summed packets then bins, in order |
| Field-light demand and intake | 19.8.1 | Per source (demand) and per receiver (intake) | Each sum in the original index order; payment |

### 5.3 Measured results (ReleaseSafe, 8 workers, 28-CPU machine)

| Scene | 19.6.5 | 19.8.0 | 19.8.1 |
|---|---:|---:|---:|
| Million-year full-pool Stellar Time | 15.5 s | 2.2 s | 2.2 s |
| Full-pool live tick (1,024 bodies, 512 packets, 64 bins) | 123 ms | 98 ms | 95 ms |
| Molten-heavy full-pool tick (every solid at 2,200 K) | about 1.5 s | 1.5 s | **0.33-0.39 s** |
| Full-pool live render frame p95, gate | 203-218 ms | n/a | 179-191 ms |

- **State agreement:** after four molten ticks, 19.8.0 and 19.8.1 at 1 and 8 workers dump
  identical bits for every body (SHA-256 `E86270AB…`).
- **Memory and save:** the Simulation is 38,340,608 bytes, an Entity 3,640 bytes, and a
  full-pool save 5,243,179 bytes.
- **Remaining cost in the molten tick:** thermal about 180 ms (pair re-booking is bounded by
  one hand-off per changed body), radiation about 125 ms and contacts about 65 ms.

---

## 6. Persistence and save formats

| Format | Release | Added |
|---|---|---|
| 26 | 19.1 | Unchanged from 18.2; view clamped to zoom 48 in files |
| 27 | 19.2 | Surface records (sparse) and unrecorded-event count |
| 28 | 19.3 | Pool ledger, active packets and bins |
| 29 | 19.4 | Loose layer, four passing marks, 512 packets |
| 30 | 19.5b | Pool thermal ledgers |
| 31 | 19.5 | Cloud columns (sparse) |
| 32 | 19.6 | Charge regions and electrical ledgers |
| 33-35 | 19.6.3 | Sparse continuation extensions; v35 bin-population ownership IDs (sharing the ID cap) |
| 36 | 19.6.4 | Authoritative momentum remainder (older formats default to zero) |
| 37 | 19.6.4 | Shared-body solid geometry and lineage (unified solids); within-capacity legacy conversion |

Rules that hold across all formats:

- Loads are validated: finite values, unique IDs plus birth ticks, and capacity.
- Older formats load without inventing material. An oversized world is rejected atomically.
- Newer saves do not open in older builds, and rolling back an executable does not make newer
  saves readable.

---

## 7. Validation and release protocol

- **Gate:** `validate.cmd` freezes a candidate manifest (hashes of src, third_party, tools and
  root build files). It then runs:
  - focused steps 17.x through 19.7;
  - probes and the cluster benchmark;
  - the launcher-guard self-test;
  - the full Debug suite;
  - the full ReleaseSafe rendering suite;
  - Debug and ReleaseSafe builds;
  - the workspace integrity probe (expected exit 3).

  Any failure stops the gate before install.
- **Populated completion** (`test-19.6.3-rebound-long`) always executes fresh
  (`has_side_effects`, 19.8.0). It covers:
  - the million-year full-pool case;
  - the 100-year thermal case;
  - memory save/resume;
  - the 19.7 combined case;
  - the 19.8 report;
  - the 19.8.1 molten agreement test.

  It also checks the REBOUND linkage evidence.
- **Install:** requires the sealed receipt, five acceptance evidence files that hash genuine
  logs, and the owner's exact line `ALLOW <VERSION> <RUN>`. The installer re-verifies hashes,
  refuses a running game, copies, and checks the installed integrity (expected 0). The
  receipt is verified again after copying.
- **Recovery points:** each part checkpoints affected sources and the installed binaries
  before its edits (`checkpoints/<version>-before-<stamp>/`).

19.8.1 final gate, with every step run fresh:

| Check | Result |
|---|---|
| Focused steps (19.5-19.7) | all 0 |
| Launcher guard | 0 |
| Full Debug | 51/51 steps, 1,383/1,388 passed, 5 skipped |
| Rendering (ReleaseSafe) | 1,238/1,239 passed, 1 skipped |
| Populated completion | 9/9, linked REBOUND |
| Debug and ReleaseSafe builds | 0 |
| Workspace / installed integrity | 3 (expected) / 0 |

Skips are recorded and are not counted as passes.

---

## 8. Approximations, exceptions and limits

Documented approximations:

- Packet-packet, packet-bin and bin-bin gravity are omitted.
- Eight broad skin sectors, a core fraction of 0.20, and 64 surface and weather columns
  (about 590 km on a 12,000-km planet).
- Water-only coarse clouds with one mean molar mass. No winds or climate model.
- Air-like entry-wall demand only. No shock radiation or full ablation.
- Breakdown only for sourced N2/O2 gas. Charge regions do not drift with transport. No
  Coulomb forces between bodies.
- Magnetosphere is the inherited proxy (no MHD).
- Finite-source shadows are approximate, with at most eight display casters.
- Coarse Stellar Time continues analytically between linked numerical windows. It is not a
  numerical integration of every orbit.
- The 19.6.5 subsurface colour is a presentation of existing heat, not a measured penetration.

Owner-approved exceptions:

- Isolated sub-1-km cleanup removes material and energy (that case only).
- Exact recorded 19.6.4 thermal-comparison and survivor-ID dispositions (phase heat about
  0.44 % against 0.3 %; cold core 0.050 % and cold skin 0.012 % against 0.01 %; one ID pair).
  These are exact measured pairs, not general tolerances.

Known limits:

- Sliding is rare at 64-column resolution.
- In Stellar Time, grazing shares land loose instead of ricocheting, and only clouds that
  overlap at a step's start collide.
- Full-pool render p95 meets the 250 ms stress limit but is not a 60 FPS promise. Diagnostic
  256-planet stress scenes run slower.
- The symmetric central-tracer IAS15 conditioning case is not qualified.
- Composite bodies are unsupported.

---

## 9. Record index

| Topic | Document |
|---|---|
| Plan, decisions and milestone status | `docs/V19_PLAN.md`, `docs/M2_PLAN.md`, `docs/M3_PLAN.md` |
| M1 | `docs/RELEASE_19_1_0.md`, `docs/RELEASE_19_2_0.md` |
| M2 | `docs/RELEASE_19_3_0.md`, `docs/RELEASE_19_4_0.md`, `RELEASE_19_4_1.md`, `RELEASE_19_4_2.md` |
| M3 | `docs/RELEASE_19_5_0.md`, `RELEASE_19_6_0.md`, `RELEASE_19_6_1.md`, `RELEASE_19_6_2.md`, `V19_6_3_PLAN.md`, `V19_6_3_PROGRESS.md`, `V19_6_3_REBOUND_ADAPTER_IMPLEMENTATION.md`, `V19_6_4_HARD_SOLID_CORRECTION.md`, `V19_6_4_LOCAL_HEATING_STAGE.md`, `RELEASE_19_6_4_20261027_035843.md`, `V19_6_5_IMPACT_HEAT_RENDERING.md`, `RELEASE_19_6_5_20261027113432.md` |
| M4 | `docs/RELEASE_19_8_0.md`, `docs/RELEASE_19_8_1.md` |
| Model review handoff | `Downloads/M3_HANDOFF_2026-10-27_19.6.5.md` |
| Change history | `CHANGELOG.md` |

Key source files:

- `physics.zig`: simulation, transactions, contacts, thermal, radiation and pools.
- `planetary.zig`: reservoirs, phase and cooling.
- `material_packets.zig`: packet and bin laws.
- `deep_time.zig`: Stellar Time.
- `rebound_backend.zig`: linked integration.
- `atmosphere_columns.zig`: weather.
- `electric.zig`: charge.
- `magnetosphere.zig`: magnetosphere proxy.
- `radiation.zig` and `shadow_geometry.zig`: light and shadow.
- `surface.zig`, `surface_view.zig`, `raster.zig` and `cloud_view.zig`: surfaces and view.
- `observation.zig`: controls.
- `parallel.zig`: worker pool.
- `savegame.zig`: persistence.
