# Kujira 20.0.3 design: the locked "UFO" observer (for owner review, no code yet)

Written 2026-11-19 after the owner tested 20.0.2 (five screenshots). Nothing here is implemented.

## 1. What I understood you want

The triangle is a ship (a UFO). Unlocked, the mouse flies it through space. Locking onto a planet means:

1. **The ship parks in the planet's rotating frame.** The planet's spin and orbit are taken out, so the planet looks
   stationary under the ship at every zoom. Panning around the planet turns the grid, and that is expected.
2. **The ship can fly anywhere over the planet**: across it, up through the atmosphere, out to the green Roche-limit
   circle, but never closer than 10 km to the ground.
3. **The camera is separate from the ship.** At deep zoom you pan the view with the mouse over the whole surface and
   the air above it. The ship stays at least 10 km up, so it is out of view and never covers the screen.
4. **Pan speed follows the zoom**, so at 0.1 m per pixel the ground doesn't race past.
5. **The Roche limit bounds the observation**: neither the ship nor the view goes beyond the green circle while locked.
6. **Nothing shakes, jumps or suddenly fills the screen.**

## 2. How 20.0.2 works now, and why it fails

| 20.0.2 rule | What you saw |
|---|---|
| The camera follows the triangle, and the side-on rotation is the triangle's angle. | The grid turns and the camera chases the triangle around the planet. |
| Spin is followed through the triangle, then the camera eases after it (a 1/12 s lag). | The planet moves with its rotation (SPEED 1,452 km/h is the planet), so the eased camera trails and wobbles. At 0.1 m per pixel a small lag is many pixels. |
| The hover height is the highest ground inside the triangle's whole 10 km footprint, plus 10 km. | The ship jumps up and down as hills enter and leave the footprint. That is shake. |
| Under a big triangle, moving the mouse up "looks up" toward the triangle, as far as its top. | Last screenshot: you moved away from the surface, the view panned up into the triangle, and it filled the screen. That is my rule breaking yours. |
| Two marks: a cursor cross, and a crosshair on the ground spot under the nose. | They do nothing useful and are confusing. |
| One mouse pixel moves the triangle one screen pixel. | The speed is right in pixels, but the camera lag and the spin made it feel fast and unstable. |

In short, I tied the camera to the triangle, then added exceptions (ground hold, look-up, hand-over weights) to stop
it covering the view. The exceptions fight each other.

## 3. The proposed model (one set of rules, no exceptions)

### 3.1 Frame

Everything locked is stored in the planet's **body frame**: polar coordinates (angle along the surface θ, height
above the centre r). θ is measured from a mark fixed on the planet, so spin is removed:

  world position = planet centre(t) + r · (cos(θ + φ(t)), sin(θ + φ(t)))

Here φ(t) is the planet's spin angle (interpolated per frame). The screen is drawn side-on: local up is screen up at
the camera's angle θ_c. The planet is then exactly stationary on screen unless the user moves. There is no easing and
no lag.

### 3.2 State

| Symbol | Meaning | Bounds |
|---|---|---|
| R(θ) | drawn ground radius under angle θ (smoothed over a few screen pixels, not 10 km) | from the surface model |
| θ_s, a_s | ship angle and altitude above ground | 10 km ≤ a_s ≤ r_Roche − R |
| θ_c, a_c | camera centre angle and altitude above ground | 0 ≤ a_c ≤ a_c,max (below) |
| m | metres per pixel at the current zoom | 375 km/px … 0.1 m/px |
| H | half the screen height in metres = (screen px / 2) · m | |

### 3.3 The ship never covers the view

The camera may look anywhere below the ship, but its top edge stays under the ship's nose:

  a_c,max = a_s − H − gap

When the ship is small on screen (H is large compared with a_s), this bound is negative, so the camera is instead
centred on the ship (a_c = a_s, θ_c = θ_s), exactly as at normal zoom today. One rule covers both:

  if a_s − H − gap ≥ 0:  camera is free in [0, a_s − H − gap]  (deep zoom: ship above the screen)
  else:                  camera = ship                         (ship visible)

The change-over is continuous, because at the boundary both rules give the same camera.

### 3.4 Mouse (relative motion dx, dy in pixels; k = sensitivity, 1 by default)

- **Ship visible** (camera = ship): the mouse flies the ship.
  Δθ_s = k · dx · m / (R + a_s),  Δa_s = −k · dy · m, then clamped to [10 km, Roche].
  The camera moves with it, so the ship stays centred and the planet turns under it.
- **Deep zoom** (ship above the screen): the mouse pans the camera.
  Δθ_c = k · dx · m / (R + a_c),  Δa_c = −k · dy · m, clamped to [0, a_c,max].
  The ship follows the camera sideways (θ_s eases to θ_c, so it is always above what you are looking at) and keeps its
  altitude. Pushing the view up against a_c,max raises the ship instead. That is how you fly up into the atmosphere at
  deep zoom without the ship entering the view.

Because one mouse pixel is m metres, panning slows exactly with zoom: 375 km per pixel when zoomed out, 0.1 m per pixel
at the floor. If that is still too fast, k can be lowered at high zoom, for example k = clamp(√(m / 100 m), 0.25, 1).

### 3.5 Zoom

A wheel step keeps the body-frame point under the cursor fixed (as free zoom does), then re-applies 3.3. Zoom out is
limited so the view's half-height stays inside the Roche circle while locked: H ≤ r_Roche.

### 3.6 Marks on screen

There is one mark only: the cursor. The ground crosshair and the second hash mark are removed.

## 4. What I need from you before coding

1. Is section 3.3 right: ship visible means the mouse flies the ship, ship above the screen means the mouse pans the
   view and the ship follows sideways?
2. Should the ship's lowest altitude be 10 km everywhere while locked, even when zoomed out?
3. Is the Roche circle the outer limit for both the ship and the zoom-out while locked?
4. Is panning one mouse pixel = one screen pixel at every zoom OK, or do you want it slower still when zoomed in
   (the k rule in 3.4)?

Version: this would be 20.0.3 (a fix inside 20.0's zoom scope), built in the development copy, tested, gated, and installed only
on your ALLOW.
