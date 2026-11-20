# Project Kujira (working title)

**A PC universe-simulation video game and full custom engine, written in Zig.**

Project Kujira is an open-world, rasterized, pixelated 2D universe under active development. It explores how matter, motion and heat can build evolving planetary and stellar systems through a bounded, interactive simulation.

This is the public **documentation and future distribution repository**. **Game and engine source code are not published here.**

## Current release: v20.0

**20.0.4 is installed in the development environment; v20.0 (the 0.1 m floor) and its four owner-tested corrections are complete.** Current save format: **40**.

**[Download the v20.0 PDF: Release Reference](PROJECT_KUJIRA_V20_0_Reference.pdf?raw=1)**

[View the PDF on GitHub](PROJECT_KUJIRA_V20_0_Reference.pdf). This 21-page reference covers the 0.1 m size floor for solids and gas, zoom to 0.1 m per pixel, the locked observer (the true 10 km triangle, free cursor and panning, zoom at the cursor, the 20 km leash, the planet held still inside its green boundary), the observer's math model, gate results and declared limits.

- [Current development status (v20.0)](docs/DEVELOPMENT_STATUS_V20.md)
- [20.0.0: the 0.1 m floor, zoom to 0.1 m per pixel](docs/RELEASE_20_0_0.md)
- [20.0.1](docs/RELEASE_20_0_1.md), [20.0.2](docs/RELEASE_20_0_2.md), [20.0.3](docs/RELEASE_20_0_3.md), [20.0.4](docs/RELEASE_20_0_4.md): the observer at deep zoom
- [The locked observer model](docs/V20_0_3_OBSERVER_MODEL.md)
- [Previous release: Universal Topology V, completed v19](UNIVERSAL_TOPOLOGY_V5_v19.pdf) and its [status](docs/DEVELOPMENT_STATUS_V19.md) and [release document](docs/RELEASE_V19_COMPLETE.md)
- [Historical 52-page atlas: Universal Topology IV, v16.3.1](UNIVERSAL_TOPOLOGY_V4_16.3.1.pdf)

| Item | Current publication |
| --- | --- |
| Project name | Project Kujira (working title) |
| Category | PC video game / universe-simulation sandbox |
| Technology | Zig; custom game and simulation engine; SDL2 / OpenGL; linked REBOUND for qualified long-time motion |
| Documented development snapshot | Installed 20.0.4; v20.0 complete; save format 40 |
| Public download | v20.0 Release Reference PDF, the v19 illustrated reference, historical atlas and release/development documentation |
| Installable game | Development installation reported locally; no public installer or executable included here |
| Game and engine source | Private; not published in this repository |

The release results are records from the completed development gate, not tests newly executed for this documentation publication. Older release notes and atlas proposals remain historical context; they do not override the current release contract.

## What the project explores

- Gravity, relative motion, contact and conserved material inventories.
- Distinct solid impacts and gas-parcel flow, drag, breakup and accretion.
- Planetary cores, layered atmospheres, visible compression and gas-giant formation.
- Stellar transformations, heat, radiation and mass-accounted emissions.
- Local orbital zones, individually tracked solid debris, accretion and supported moon/ring outcomes.
- Universe time, body ages, orbital clocks, navigation and observation.
- Pixelated visualization of composition, temperature, illumination and scale.

The engine combines physical relationships with explicit numerical and gameplay approximations. It is a game under development, not a validated scientific prediction tool or safety-critical application. The atlas distinguishes implemented mechanics, calculated examples, reported checks and proposed work.

## Development and release direction

The current documented development release is installed 20.0.4, with v20.0 complete. A gated local development installation is not a public distribution package. Release timing, system requirements and packaging for a public installable game will be announced separately; this update includes no installer.

Future updates may include revised atlases, development notes and downloadable game packages. This repository is not a commitment to publish the game's source code.

## Licensing

The intended software-license direction for a future installable release is the **[MIT License](https://opensource.org/license/mit)**, with the applicable copyright and third-party notices included in that release. MIT is a license, not a registration or certification.

That future software-license plan does **not** automatically relicense the current PDF, illustrations, project identity or other documentation. See [Licensing and publication scope](LICENSING.md).

## Software quality and safety direction

Development aims to use documented verification, validation, regression testing and release checks, informed by relevant IEEE software-engineering guidance, including **[IEEE 1012-2024: System, Software, and Hardware Verification and Validation](https://standards.ieee.org/ieee/1012/7324/)**.

Planned release practices include testing installation and updates, preserving save compatibility, documenting known limitations, checking performance budgets, and considering visual comfort around bright or rapidly changing effects.

**No IEEE certification, independent safety assessment, formal standards conformance or endorsement is claimed.** These are development objectives, not completed certifications.

## Creator

- GitHub: [@kujiwa-rgb](https://github.com/kujiwa-rgb)
- X: [@officialemane](https://x.com/officialemane)

Copyright 2026 the project creator, identified above. Project Kujira is a working title.
