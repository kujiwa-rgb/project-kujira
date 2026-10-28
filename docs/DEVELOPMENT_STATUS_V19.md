# Project Kujira: current development stage

As of **October 28, 2026**. Public documentation update only.

## Release position

- **19.8.1** is reported installed and v19 milestones **M1-M4** are complete.
- Save format **37**; the v19 release history covers formats 26-37.
- The recorded installed executable SHA-256 is `24D7185D3EC67231523592826B8BB5235EA5A80C9D1730577CF6665FD3878B76`.
- The recorded final gate is `19.8.1/validation-final-20261027231328`.
- The guard sealed the receipt and authorized installation; installed integrity was reported passing. These are existing development records, not newly run checks.
- A rollback copy of 19.8.0 is retained locally. No rollback files are published here.

## What v19 established

Persistent planetary surface/history records; progressive raster refinement; Tab focus locking and triangle-centered steering; shared physical-solid ownership and hard contacts; conservative funded products; thermal/phase and atmospheric accounting; local entry/impact heat and tapered subsurface presentation; charge inventories and gas-only lightning; the inherited magnetosphere proxy; finite-source lighting; and a coarse, resumable Stellar Time path with analytic and linked REBOUND motion.

The physical capacities remain distinct: **1,024 shared main-body slots**, **512 gas/material packets**, and **64 bins with up to four gas/material modes**. The retired independent 256-solid allocator is not the current architecture. All independently positioned solid products use the shared body pool.

New physical solid pieces must be at least **1 km in diameter**, funded from their actual retained volume. A separate, owner-approved existing sub-1-km cleanup can remove owned material and energy; this narrow gameplay exception is not conservative vaporization.

## Recorded qualification and performance

- Full Debug: **51/51 steps**, **1,383/1,388 tests passing**, **5 skipped**.
- Rendering: **1,238/1,239 passing**, **1 skipped**.
- Fresh completion acceptance: **9/9**; actual Debug and ReleaseSafe engine builds passed.
- Full-pool million-year-plus-2.5-second skip: **2.1842 seconds**, 14 accepted windows, zero retries, exact requested endpoint.
- Combined million-year case: **3.3727 seconds**, 15 accepted windows, one merger, seven events and zero unresolved events.
- Final molten full-pool benchmark: approximately **1,510.99 ms** per tick with one worker versus **336.66 ms** with eight workers, about **4.49x** for that measured scene.
- Full-pool rendered-frame p95 remained approximately **179-191 ms**, within the recorded **250-ms stress budget**. This is not a universal 60-FPS guarantee.

Different benchmark scenes and metric types are not interchangeable. Skipped tests are not passes. Reported one/eight-worker agreement applies to the recorded fixtures; it is not a proof for every possible scene or future thread schedule.

## Explicit boundaries

The engine is a game simulation, not a calibrated astrophysical research solver. Long skips deliberately approximate quiet evolution and encounter history; they do not replay every local weather effect or numerical orbit. Packet/bin mutual gravity, the simplified atmosphere, entry-heating proxy and presentation-estimated subsurface heat depth have declared limits.

The release retains specific documented comparison exceptions. Those exceptions do not increase general conservation tolerances or turn every original failed assertion into a passing result. See the complete release document and PDF for the exact scope.

## Read the current references

- [Illustrated v19 engine reference](../UNIVERSAL_TOPOLOGY_V5_v19.pdf)
- [Complete v19 release](RELEASE_V19_COMPLETE.md)
- [19.8.0 review and acceleration](RELEASE_19_8_0.md)
- [19.8.1 exact threading](RELEASE_19_8_1.md)

## Publication scope

This repository remains **documentation-only**. No engine/game source, executable, installer, player save, compiler cache, temporary test world, raw gate bundle or rollback checkpoint is included. Source paths and local gate identifiers appearing in the release notes identify private development records; their presence does not publish those files.

The existing [licensing and publication scope](../LICENSING.md) remains in force. This update does not apply a new software or documentation license. No new engine tests or builds were run to publish this snapshot.
