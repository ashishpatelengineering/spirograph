# Spirograph

A browser-based spirograph you can shape by hand. Drag the wheel and pen centers directly on the canvas, then press play to draw hypotrochoid curves, layer patterns on top of each other, and export the result as a PNG.

**Live demo:** [Spirograph](https://ashishpatelengineering.github.io/spirograph/)

---

## Features

- **Drag-to-shape**: grab the wheel's center or the pen's center right on the canvas to resize the wheel or move the pen. Sliders are also there for precise or keyboard-friendly input.
- **Play / Pause / Stop**: standard transport controls. Stop rewinds without erasing your drawing.
- **Layering**: adjusting the shape or pressing Play again draws a new pattern *on top* of the old one, so you can build up layered designs. A separate Clear button wipes the canvas when you actually want a blank page.
- **7 pen colors**: 6 curated presets plus a custom color picker slot.
- **Download PNG**: save your pattern as an image.
- **Math tab**: a full written explanation of the equations behind the drawing, with LaTeX-rendered formulas and references.

No build step, no dependencies to install. It's a single HTML file that runs entirely in the browser.

---

## How to use it

1. Open `index.html` in any modern browser.
2. Drag the grey dot (wheel center) in or out to change the wheel's size.
3. Drag the colored dot (pen) to change how far the pen sits from the wheel's center.
4. Press **Play**. The pattern draws itself and stops automatically once it closes into a finished shape.
5. Change the pen color and adjust the shape again to layer a new pattern on top, or hit **Clear** to start fresh.

---

## How it works

Everything lives in one file, `index.html`: CSS at the top, HTML markup in the middle, and all the JavaScript logic at the bottom in a single `<script>` tag.

### The core idea

Picture a small wheel rolling around the inside of a bigger fixed ring, with a pen attached to the wheel at some distance from its center. As the wheel rolls, the pen draws a curve. That's the whole app: everything else is just controls for that one mechanism.

Three numbers describe the mechanism at any moment:

| Variable | What it means | In the code |
|---|---|---|
| `R` | Radius of the fixed outer ring | Always `1` |
| `r` | Radius of the rolling wheel | `state.r` |
| `d` | Distance from the wheel's center to the pen | `state.d` |
| `t` | The angle the wheel has rotated around the ring so far | `state.t` |

### The math

The pen's position at angle `t` comes from one function, `penPoint(t)`:

```js
function penPoint(t) {
  const cx = (R - r) * Math.cos(t);           // wheel's center, x
  const cy = (R - r) * Math.sin(t);           // wheel's center, y
  const spin = (R - r) / r * t;               // how fast the pen spins around that center
  const x = cx + d * Math.cos(spin);          // pen position, x
  const y = cy - d * Math.sin(spin);          // pen position, y
  return { x, y, cx, cy, spin };
}
```

Find where the wheel's center is right now, then add the pen's offset from that center. The pen spins around that center faster than the center itself travels around the ring. That spin rate, `(R - r) / r`, isn't arbitrary: it follows directly from the wheel rolling without slipping.

### Drawing the curve

The app doesn't draw the whole curve at once. Every animation frame, it nudges `t` forward by a small step and draws a short straight line from the old pen position to the new one:

```js
function drawInkSegment(t0, t1) {
  const p0 = lastPoint || penPoint(t0);
  const p1 = penPoint(t1);
  // draw a line from p0 to p1 on the canvas
  lastPoint = p1;
}
```

Do this fast enough, many times per second, and the short straight lines look like one smooth curve.

### Knowing when to stop

The curve eventually loops back to exactly where it started. The app figures out *when* that happens using basic number theory: it writes `r` as a fraction in lowest terms (like 0.25 = 1/4), and the reduced numerator tells it how many times the wheel needs to go around the ring before the pattern closes:

```js
function computeGeometry() {
  const rInt = Math.round(state.r * 100);   // r as a whole number out of 100
  const g = gcd(100, rInt);                 // reduce the fraction
  const loops = rInt / g;                   // how many trips around the ring
  state.tMax = 2 * Math.PI * loops;         // stop once t reaches this
}
```

This is also why `r` gets rounded to the nearest 0.01 whenever you drag it: it keeps this calculation and the actual drawing perfectly in sync, so the curve always closes cleanly instead of trailing off.

### Dragging the wheel and pen

The wheel's center always sits at a fixed distance, `R - r`, from the ring's center, no matter which way it's rotated. So dragging just measures how far your cursor is from the center and uses that distance to set `r`. The pen handle works the same way, but measured from the wheel's center instead, to set `d`.

### Layering patterns

Nothing about resizing the wheel, moving the pen, or pressing Play again erases the canvas. Only the dedicated **Clear** button does that. This is what lets you build up multiple patterns on the same drawing.

### The Math tab

A second view (`#view-math`) sits alongside the drawing canvas. It's a static write-up of the equations above, rendered with [KaTeX](https://katex.org/) (loaded from a CDN), covering where the equations come from, when the curve closes, and references for further reading.

---

## File structure

```
index.html   ← everything: styles, markup, and logic in one file
```

There's nothing to install and nothing to build. Just open the file, or serve the folder with GitHub Pages.

---

## Credits

Built with plain HTML, CSS, and JavaScript. Equations rendered with [KaTeX](https://katex.org/). See the in-app Math tab for mathematical references (Wolfram MathWorld, Wikipedia) and a short history of the Spirograph toy.
