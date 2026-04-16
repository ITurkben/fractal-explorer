# Fractal Explorer

An interactive, GPU-accelerated fractal explorer — four fractals, six coloring modes, deep zoom with emulated double precision, guided tour, and video recording. No frameworks, no build step. Just a single HTML file.

**[Live Demo on GitHub Pages](https://iturkben.github.io/fractal-explorer/)**

---

## Features

| Category | Details |
|---|---|
| **Fractals** | Mandelbrot, Julia, Burning Ship, Newton (z³−1) |
| **Coloring modes** | Smooth iteration, Distance Estimation, Orbit Trap, Interior Coloring |
| **Color palettes** | 6 palettes: Fire, Ocean, Psychedelic, B&W, Rainbow, Neon |
| **Deep zoom** | Veltkamp-Dekker emulated double-float in GLSL — ~14 digits of precision |
| **Anti-aliasing** | 2×2 supersampling per pixel (toggle) |
| **Screenshot** | Download PNG at 2× or 4× viewport resolution (offscreen framebuffer) |
| **Guided Tour** | 8 legendary locations with smooth animated fly-to (logarithmic zoom interpolation) |
| **Video Recording** | Zoom animation captured via `MediaRecorder` → download `.webm` |
| **Shareable URLs** | Full state encoded as query parameters |
| **Intro animation** | Automatic zoom into Seahorse Valley on first load |
| **Touch support** | Pinch-to-zoom and drag-to-pan on mobile |
| **Keyboard shortcuts** | Full navigation and control via keyboard (see below) |
| **localStorage** | Palette, AA, coloring mode, and iterations persist across reloads |
| **Auto-hide UI** | Controls fade out after 3 s of mouse inactivity |

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `↑ ↓ ← →` | Pan view |
| `+` / `-` | Zoom in / out |
| `R` | Reset view |
| `F` | Toggle fullscreen |
| `H` | Hide / show UI |
| `A` | Toggle anti-aliasing |
| `P` | Cycle color palette |
| `1`–`4` | Switch fractal type |
| `I` | Toggle FPS counter |
| `S` | Screenshot (2×) |
| `U` | Copy share URL |
| `T` | Open / close guided tour |
| `?` | Show keyboard help |
| `Esc` | Close panels / cancel animation |
| `Alt` + mouse move | Set Julia c parameter live |

---

## Featured Locations

### Seahorse Valley — Mandelbrot
Center: `−0.7436438870371587 + 0.1318259042i` · Zoom: ~4500×

Intricate spiral arms resembling seahorse tails emerge from the junction between the main cardioid and the period-2 bulb. One of the most photographed regions of the Mandelbrot set.

### Elephant Valley — Mandelbrot
Center: `0.275 + 0.006i` · Zoom: ~1800×

Large, ordered circular bulbs line up in a procession reminiscent of elephants, decreasing in size towards a central point.

### Triple Spiral Valley — Mandelbrot
Center: `−0.088 + 0.654i` · Zoom: ~1600×

Three spirals interweave in a hypnotic choreography, each harboring its own sub-copies of the Mandelbrot set.

### Mini Mandelbrot — Mandelbrot
Center: `−1.75 + 0i` · Zoom: ~600×

A perfect miniature copy of the entire Mandelbrot set embedded within itself — a canonical example of self-similarity. Infinitely many such copies exist at every scale.

### Julia Dragon — Julia, c = −0.8 + 0.156i
A fire-breathing dragon emerges from the Julia set at this parameter — one of the most visually dramatic connected Julia sets.

### Julia Dendrite — Julia, c = 0 + i
At `c = i`, the Julia set sits precisely at the boundary between connected and disconnected, forming a perfectly symmetric snowflake-like dendrite.

### Burning Ship Mast — Burning Ship
Center: `−1.775 − 0.005i` · Zoom: ~1400×

The iconic "mast" structure of the Burning Ship fractal. The abs() in the iteration formula creates asymmetric, ship-like forms unique to this fractal.

### Newton Classic — Newton (z³−1)
The complete Newton fractal for `f(z) = z³ − 1`, revealing three interlocking color basins corresponding to the three cube roots of unity.

---

## Controls

| Action | Desktop | Mobile |
|---|---|---|
| Zoom | Scroll wheel | Pinch |
| Pan | Click + drag | Drag |
| Center on point | Double-click | — |
| Julia c (live) | Hold `Alt` + move mouse | Sliders |
| Cancel animation | Click, scroll, or `Esc` | Touch |

---

## The Mathematics

### Mandelbrot Set

The Mandelbrot set is the set of complex numbers **c** for which the sequence

> z₀ = 0,   z_{n+1} = z_n² + c

remains **bounded**. Points inside are black; points outside are colored by how quickly they escape, using the smooth (normalized) iteration count formula:

> μ = n + 1 − log₂(log₂|z_n|)

This removes visible "staircase" banding from raw iteration counts.

### Julia Sets

Uses the same iteration `z_{n+1} = z_n² + c` with **c fixed** and `z₀ = pixel coordinate`. Each choice of c produces a unique shape — ranging from connected filled regions to totally disconnected dust. The Julia set boundary is fractal whenever c lies on the Mandelbrot boundary.

### Burning Ship

> z_{n+1} = (|Re(z_n)| + i·|Im(z_n)|)² + c

The `abs()` operation "folds" the plane before each squaring, creating the asymmetric ship-like forms. This same asymmetry makes the Distance Estimation derivative discontinuous at fold lines (see Technical Notes).

### Newton Fractal

Applies Newton's root-finding iteration to **f(z) = z³ − 1**:

> z_{n+1} = z_n − f(z_n)/f′(z_n) = (2z_n³ + 1) / (3z_n²)

The three roots of z³ = 1 are `1`, `ω = e^(2πi/3)`, and `ω² = e^(4πi/3)`. Each pixel is colored by which root the iteration converges to (blue/green/red) and how quickly (brightness). The basin boundaries exhibit fractal structure with infinite complexity.

---

## Technical Notes

### Double-Float Precision (Veltkamp-Dekker)

Standard single-precision `float` has ~7 significant decimal digits, limiting useful zoom depth to roughly 10⁷. To reach deeper zoom levels without pixelation, each complex coordinate is represented as a pair of floats `(hi, lo)` where the true value ≈ `hi + lo`.

All arithmetic on these pairs uses the **Veltkamp-Dekker** algorithm:

```glsl
// Add two double-floats
vec2 df_add(vec2 a, vec2 b) {
  float s = a.x + b.x, v = s - a.x;
  float e = (a.x-(s-v)) + (b.x-v) + a.y + b.y;
  float hi = s + e;
  return vec2(hi, e - (hi - s));
}

// Multiply two double-floats using the Veltkamp split (SPLIT = 2^12 + 1 = 4097)
vec2 df_two_prod(float a, float b) {
  float p = a * b;
  const float S = 4097.0;
  float at=S*a, ah=at-(at-a), al=a-ah;
  float bt=S*b, bh=bt-(bt-b), bl=b-bh;
  return vec2(p, ((ah*bh-p)+ah*bl+al*bh)+al*bl);
}
```

On the JavaScript side, the center coordinates are regular 64-bit doubles. They are split into `(hi, lo)` pairs via `Math.fround()` before being passed as uniform `vec2` values to the shader. This gives ~14 significant digits — equivalent to IEEE 754 `double` precision.

### Distance Estimation (DE)

For escape-time fractals, alongside the orbit `z_n`, we track the **derivative** `dz_n/dc` (for Mandelbrot) or `dz_n/dz_0` (for Julia):

```
dz_{n+1}/dc  = 2·z_n · (dz_n/dc) + 1   (Mandelbrot)
dz_{n+1}/dz₀ = 2·z_n · (dz_n/dz₀)      (Julia)
```

When the orbit escapes at iteration `n`, the estimated distance from `c` to the nearest boundary point is:

> d(c) ≈ |z_n| · log|z_n| / |dz_n|

This distance is converted to a "glow" value `exp(−d·zoom·k)` that is bright (≈1) near the boundary and dark away from it. The result is thin, luminous boundary lines rather than filled color bands.

**Limitation for Burning Ship:** The `abs()` operation creates cusps in the Julia set where the mathematical derivative is discontinuous. The DE computation for Burning Ship uses a sign-based approximation that ignores these discontinuities — results near fold lines may show artifacts. The visualization is still useful for boundary detection but is not mathematically exact.

**Not applicable to Newton:** The Newton fractal converges rather than escapes; there is no escape-time distance estimate. The standard root-colored visualization is used instead.

### Orbit Trap Coloring

Instead of coloring by escape time, we track the **minimum distance** from the orbit to a "trap" shape. This explorer uses a **cross trap** — the minimum of `|Re(z)|` and `|Im(z)|` (proximity to either axis):

```glsl
trapMin = min(trapMin, min(abs(zrF), abs(ziF)));
```

The final color is `palette(exp(−trapMin × k))`, which is brightest where the orbit passed closest to the axes. Cross traps tend to produce diamond/square-like geometric patterns that reveal the periodic structure of orbits near the boundary.

### Interior Coloring

By default, points inside the Mandelbrot set (orbits that never escape) are rendered black. Interior coloring assigns each inside pixel a color based on the **angle of the final orbit position** after `maxIter` iterations:

```glsl
float angle = atan(ziF, zrF) / (2π) + 0.5;
return palette(angle + |z| × 0.08) × 0.65;
```

This reveals the internal "veins" and symmetric structure within the Mandelbrot set that is otherwise invisible.

---

## Tech Stack

- **WebGL 2 / GLSL ES 3.00** — GPU rendering (mandatory for real-time performance)
- **Vanilla JavaScript** — no frameworks, no dependencies, no build step
- **Single `index.html`** — self-contained, deployable anywhere
- **GitHub Actions** — automatic deployment to GitHub Pages on every push to `main`
- **MediaRecorder API** — in-browser video capture (Chrome/Edge; limited Firefox support)

### Browser Support

| Feature | Chrome | Firefox | Safari | Edge |
|---|---|---|---|---|
| WebGL 2 | ✅ | ✅ | ✅ (15+) | ✅ |
| Double-float zoom | ✅ | ✅ | ✅ | ✅ |
| Video recording | ✅ | ⚠️ (no VP9) | ❌ | ✅ |
| Screenshot (PNG) | ✅ | ✅ | ✅ | ✅ |

---

## Local Development

No build step required:

```bash
npx serve .
# or
python -m http.server 8080
# or just open index.html in a browser
```

---

## Shareable URLs

Click **Share** or press `U` to copy a link encoding the current view. All parameters are preserved:

| Param | Meaning |
|---|---|
| `cx`, `cy` | Center of view (complex plane coordinates) |
| `zoom` | Zoom level (pixels per complex unit) |
| `type` | Fractal (0=Mandelbrot, 1=Julia, 2=BurningShip, 3=Newton) |
| `jcre`, `jcim` | Julia constant c = jcre + jcim·i |
| `iter` | Max iterations (50–2000) |
| `pal` | Color palette (0–5) |
| `coff` | Color offset (0–1) |
| `aa` | Anti-aliasing (0 or 1) |
| `cm` | Coloring mode (0=smooth, 1=DE, 2=orbitTrap, 3=interior) |

---

## License

MIT — screenshots, videos, and forks are welcome.
