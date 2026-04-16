# Fractal Explorer

An interactive, GPU-accelerated fractal explorer built with vanilla JavaScript and WebGL 2. Explore breathtaking mathematical structures in real time — right in your browser.

**[Live Demo on GitHub Pages](https://iturkben.github.io/fractal-explorer/)**

---

## Features

| Feature | Details |
|---|---|
| **4 Fractals** | Mandelbrot, Julia, Burning Ship, Newton |
| **Smooth coloring** | Normalized iteration count — zero banding |
| **6 Color palettes** | Fire, Ocean, Psychedelic, B&W, Rainbow, Neon |
| **Deep zoom** | Emulated double-float precision in GLSL |
| **Anti-aliasing** | 2× supersampling (toggle) |
| **Screenshot** | Download PNG at 2× or 4× viewport resolution |
| **Shareable URLs** | Full state encoded as query params |
| **Intro animation** | Auto-zooms into Seahorse Valley on load |
| **Touch support** | Pinch-to-zoom and drag-to-pan on mobile |
| **Auto-hide UI** | Controls fade after 3 s of inactivity |

---

## Controls

| Action | Desktop | Mobile |
|---|---|---|
| Zoom | Scroll wheel | Pinch |
| Pan | Click + drag | Drag |
| Center on point | Double-click | — |
| Julia c (live) | Hold **Alt** + move mouse | Sliders |
| Show UI | Move mouse | Touch screen |

---

## The Mathematics

### Mandelbrot Set

The Mandelbrot set is the set of complex numbers **c** for which the sequence

> z₀ = 0,   z_{n+1} = z_n² + c

remains **bounded** (does not escape to infinity). Points inside the set are colored black; points outside are colored by how quickly they escape — creating the iconic infinitely detailed boundary.

**Smooth coloring:** instead of counting discrete iterations, we use the normalized escape count:

> μ = n + 1 − log₂(log₂|z_n|)

This removes the visible "staircase" banding of raw iteration counts.

### Julia Sets

A Julia set uses the same iteration **z_{n+1} = z_n² + c**, but **c is a fixed constant** and the initial value **z₀ = pixel coordinate**. Each choice of c produces a different shape — ranging from connected "dendrites" to totally disconnected "dust." The two sliders (or Alt+mouse) let you explore this parameter space in real time.

### Burning Ship

A variation of Mandelbrot where the absolute values of the real and imaginary parts are taken before each squaring:

> z_{n+1} = (|Re(z_n)| + i|Im(z_n)|)² + c

The result looks like a burning ship viewed from below, with self-similar flames along the hull.

### Newton Fractal

Applies Newton's root-finding method to **f(z) = z³ − 1**:

> z_{n+1} = z_n − f(z_n)/f′(z_n) = (2z_n³ + 1) / (3z_n²)

The three roots of z³ = 1 are:
- **1**
- **ω = e^(2πi/3) ≈ −0.5 + 0.866i**
- **ω² = e^(4πi/3) ≈ −0.5 − 0.866i**

Each pixel is colored by which root the iteration converges to (blue / green / red) and how many iterations it took (brightness). The resulting fractal is a mesmerizing trichromatic basin boundary.

---

## Deep Zoom & Precision

Standard single-precision floats have ~7 significant decimal digits — enough for zoom levels up to ~10⁷ before pixelation appears. To support deeper exploration, this explorer implements **double-float emulation** directly in GLSL:

Each coordinate is represented as **two 32-bit floats** `(hi, lo)` such that the true value ≈ hi + lo. Arithmetic (Veltkamp-Dekker algorithm) preserves this invariant through addition and multiplication, yielding ~14 significant digits — matching IEEE 754 double precision on the CPU.

---

## Tech Stack

- **WebGL 2 / GLSL ES 3.00** — GPU rendering (mandatory; CPU would be too slow)
- **Vanilla JS** — zero frameworks, zero dependencies
- **Single `index.html`** — no build step required
- **GitHub Actions** — automatic deploy to GitHub Pages on every push to `main`

---

## Local Development

```bash
# Any static server works — no build step needed
npx serve .
# or
python -m http.server 8080
```

Then open `http://localhost:8080`.

---

## Shareable URLs

Click **Share URL** to copy a link encoding the current view. Parameters:

| Param | Meaning |
|---|---|
| `cx`, `cy` | Center of view (complex plane) |
| `zoom` | Zoom level (pixels per unit) |
| `type` | Fractal (0–3) |
| `jcre`, `jcim` | Julia constant c = jcre + jcim·i |
| `iter` | Max iterations |
| `pal` | Color palette (0–5) |
| `coff` | Color offset |
| `aa` | Anti-aliasing (0/1) |

---

## License

MIT — do whatever you want with it. Screenshots welcome!
