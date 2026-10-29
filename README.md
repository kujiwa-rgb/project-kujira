# Project Kujira (working title)

**A PC universe-simulation video game and full custom engine, written in Zig.**

Project Kujira is an open-world, rasterized, pixelated 2D universe under active development. It explores how matter, motion and heat can build evolving planetary and stellar systems through a bounded, interactive simulation.

This is the public **documentation and future distribution repository**. **Game and engine source code are not published here.**

## Current release: v19 complete

**19.9.1 is installed in the development environment; v19 milestones M1-M4 and the 19.9 fixes are complete.** Current save format: **37**.

**[Download the new PDF: Universal Topology V, completed v19](UNIVERSAL_TOPOLOGY_V5_v19.pdf?raw=1)**

[View the PDF on GitHub](UNIVERSAL_TOPOLOGY_V5_v19.pdf). This 40-page illustrated reference covers the engine models and equations, milestone history, thermal and material accounting, Stellar Time and REBOUND, surfaces and controls, exact parallel work, measured results and declared limits.

- [Current development status and publication scope](docs/DEVELOPMENT_STATUS_V19.md)
- [Complete v19 release document](docs/RELEASE_V19_COMPLETE.md)
- [19.9.1 physical impact debris and installed closeout](docs/RELEASE_19_9_1.md)
- [19.9.0 vapour, clouds and directional debris](docs/RELEASE_19_9_0.md)
- [19.8.0 release record](docs/RELEASE_19_8_0.md)
- [19.8.1 exact-threading release record](docs/RELEASE_19_8_1.md)
- [Historical 52-page atlas: Universal Topology IV, v16.3.1](UNIVERSAL_TOPOLOGY_V4_16.3.1.pdf)

| Item | Current publication |
| --- | --- |
| Project name | Project Kujira (working title) |
| Category | PC video game / universe-simulation sandbox |
| Technology | Zig; custom game and simulation engine; SDL2 / OpenGL; linked REBOUND for qualified long-time motion |
| Documented development snapshot | Installed 19.9.1; v19 complete; save format 37 |
| Public download | Illustrated v19 PDF, historical atlas and release/development documentation |
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

The current documented development release is installed 19.9.1, with v19 complete. A gated local development installation is not a public distribution package. Release timing, system requirements and packaging for a public installable game will be announced separately; this update includes no installer.

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
