# Fourier Outline Tracer

**Draw anything with rotating circles.** Fourier Outline Tracer turns text, photos, freehand sketches and mathematical equations into a single machine of rotating circles (epicycles) that redraws the shape, using real Fourier series computed in the browser. It runs entirely client-side as one self-contained HTML page: no server, no account, no upload of your images anywhere.

It started from a short math video showing a chess knight drawn by two chains of circles, and grew into a full instrument for exploring how any outline can be written as a sum of circles, in 2D, 3D and 4D (space plus time).

---

## Table of contents

1. [What it does](#what-it-does)
2. [Feature list](#feature-list)
3. [The mathematics](#the-mathematics)
4. [How a shape becomes circles (pipeline)](#how-a-shape-becomes-circles-pipeline)
5. [3D and 4D](#3d-and-4d)
6. [Accuracy and the error guarantee](#accuracy-and-the-error-guarantee)
7. [Rendering and precision](#rendering-and-precision)
8. [Technology](#technology)
9. [Project files](#project-files)
10. [Browser support](#browser-support)
11. [Privacy](#privacy)
12. [Known limits](#known-limits)
13. [Methods and references](#methods-and-references)
14. [Credits and licences](#credits-and-licences)
15. [Related documents](#related-documents)

---

## What it does

Give it a shape, from any of four sources:

| Source | What happens |
|---|---|
| **Text** | Your words are turned into exact vector outlines straight from the font file (Bézier curves), so letters stay sharp at any zoom. |
| **Image** | A photo or drawing is split into its main colours; the edge of every colour region is traced to sub-pixel accuracy. |
| **Draw** | You sketch on the canvas with mouse, pen or finger; each stroke becomes an outline. |
| **Equation** | You paste or type any parametric equation (x(t), y(t)), including equations exported by this app, Desmos, GeoGebra, Mathematica or Python, and it draws by itself a moment later. |

The app then:

1. Joins every outline into **one closed path**, with pen-up travel moves between shapes.
2. Computes the **Fourier series** of that path with a fast Fourier transform.
3. Builds a **chain of rotating circles**: one circle per Fourier coefficient, each riding on the tip of the previous one, largest first. The main circle stays fixed for the whole drawing.
4. **Animates** the chain: the pen at the end of the chain traces the drawing, outline by outline, and filled colours appear as each shape is completed.
5. Lets you **export** the result as a vector SVG, a 4K PNG, a WebM video, the raw coefficients (JSON), or as a full **equation** in five formats.

---

## Feature list

### Sources
- Text with 5 built-in open-licence heavy fonts (Archivo Black, Anton, Playfair Display Black, Rubik Black, Bricolage Grotesque ExtraBold), plus **your own font** (TTF, OTF or WOFF).
- Automatic line wrapping for long sentences; up to 24 lines.
- One colour per letter, or a single colour of your choice.
- Image upload by button, drag and drop, or paste; PNG, JPG, WebP, GIF, SVG, and HEIC where the browser can decode it.
- Freehand drawing with on-canvas **pen, eraser, undo and clear** tools. Closed strokes fill; open strokes draw as lines. The playback bar's play button turns the strokes into circles.
- Six **ready-made drawings** (Horse, Garden, Sunset, Flower, Butterfly, Sailboat), loaded as editable strokes.
- Equation simulator that draws automatically as you paste or type, with 9 examples (a real rose built from 7 curves, heart, butterfly curve, spirograph, starfish, trefoil knot, cardioid, nephroid, Lissajous) and automatic detection of the parameter range.

### Machines
- **Circles in circles**: the complex form, z(t) = Σ cₖ e^{ikt}, one circle per coefficient, sorted by radius.
- **X / Y guides**: the orthogonal form from the original video, one chain sets x, one sets y, and the pen sits where their guide rods cross (elliptic Fourier descriptors). Every harmonic of both machines is summed in double precision, so their innermost circles are visible at any zoom and "Zoom into the last circle" can follow either machine.
- **One fixed machine** (default): every outline on one path, one series, the main circle never moves.
- **Machine per outline**: each outline gets its own small machine.
- **2D, 3D and 4D** modes (see below).

### Quality
- **Draft / HD / Ultra HD** presets: the largest allowed gap between the circles' curve and the true outline is 0.5 %, 0.1 % or 0.025 % of the drawing height. The app finds the fewest circles that meet it.
- **Manual**: choose the number of circles yourself (logarithmic slider, up to 100,000 on one machine).
- Optional **Lanczos σ damping** to reduce ringing (Gibbs overshoot) at sharp corners.
- Live readouts: widest gap, gap at fixed N values, the mathematical upper bound, mean gap, outline and sample counts.

### Exploration
- **Zoom** from 0.1 % to effectively unlimited, with sharp vector rendering at every level; works in both circle styles.
- **Zoom into the last circle**: the camera flies onto the innermost circle at the pen and follows it, slowing time so you can see it turn.
- **Microscope**: dive down the chain from the largest circle to the smallest, with time dilation and a depth readout.
- **Watch it converge**: the series grows live from 1 circle to thousands while the error shrinks.
- **Hear the shape**: the outline played as stereo sound (x left, y right), the way oscilloscope art works.
- **Compare with original**: your photo and the drawing side by side, zooming and panning together.
- **Direction arrow** ahead of the pen; dashed target outlines that stay on top until each shape is finished.
- **Neon glow** style, background colour presets plus a custom picker, grid on/off.
- **Spectrum** chart of the circle radii in use and left out.

### Playback
- Play, pause, restart, finish, **back / forward 5 %**, and a draggable **progress slider**.
- Speed from **0.03×** to **16×**.
- On phones the bar is one compact row: play/pause icon, progress slider, restart icon, percentage, and a speed button that opens a small pop-up.

### Fill styles
- **Solid colour**, **Drawn lines** (hatching with adjustable line count, thickness, angle, cross-hatch), or **Outline only**.

### Export
- **SVG**: smooth cubic Bézier curves (one path per colour), optional background.
- **PNG 4K**: longest side 4096 px.
- **Video**: WebM recording of the canvas.
- **Coefficients**: JSON list of every circle (frequency k, real and imaginary part).
- **Equation**: the full Fourier equation in five formats, ready to paste into Desmos, GeoGebra, Mathematica, Python, or back into this app. In 3D it includes z(t) (Desmos 3D, GeoGebra 3D, ParametricPlot3D, Matplotlib 3D), and pasting it back rebuilds the 3D object. In 4D it lists every shape of the cycle plus the blend with a τ slider.
- **3D model** (3D mode): **GLB** (glTF 2.0 binary with colours, or a vertex-coloured depth surface for photos) and **STL** (millimetres, longest side 100 mm, zero-area triangles removed; a few open edges can remain where tight curves fold, so let the slicer repair it).
- **Looping GIF** (4D mode): 48 frames, one full period of τ and one full camera turn.

### Interface
- Responsive layout: canvas and controls side by side on desktop. On phones and small tablets the canvas fills the screen and all settings move into a **☰ slide-in menu** (close with ✕, a tap outside, a swipe or Esc).
- **Full screen** button, and a one-time prompt on phones in portrait that goes full screen and turns the screen to landscape where the browser allows it.
- Touch: pinch to zoom, one finger to pan, double-tap to zoom in; in 3D/4D one finger turns the object (it follows your finger: drag right turns right, drag down tilts down) and two fingers move it. Buttons are at least 44 px tall on touch screens.
- Light and dark themes follow the system setting.
- Interface text cannot be selected or copied; text fields stay editable.

---

## The mathematics

### A closed curve as a sum of circles

Treat each point of a closed outline as a complex number z = x + iy and walk around it once as t goes from 0 to 2π. Any such curve can be written as a Fourier series:

```
z(t) = Σ  cₖ · e^{i k t}        (k = …, −2, −1, 0, 1, 2, …)
```

Each term cₖ e^{ikt} is a vector of length |cₖ| that turns k times per lap, anticlockwise for positive k, clockwise for negative k. Put the vectors tip to tail and the last tip draws the curve. That chain of arms, each sweeping out a circle, is the drawing machine.

The coefficients come from the discrete Fourier transform of M evenly spaced samples z₀ … z_{M−1}:

```
cₖ = (1/M) Σⱼ zⱼ · e^{−i k 2π j / M}
```

### The orthogonal (x / y) form

Each coordinate also has its own real Fourier series:

```
x(t) = a₀ + Σ ( aₖ cos kt + bₖ sin kt )
y(t) = c₀ + Σ ( cₖ cos kt + dₖ sin kt )
```

A cosine and sine of the same frequency combine into one arm of length √(aₖ² + bₖ²) turning k times per lap. One chain sets x, one sets y, and the pen sits where their guides cross. With one harmonic per axis (N = 1) both coordinates are pure sinusoids, so the curve is an ellipse. This is the elliptic Fourier descriptor form (Kuhl and Giardina, 1982).

The two forms are the same series. With x and y packed into one complex transform C = X + iY, the real coefficients are:

```
aₓₖ = Re Cₖ + Re C₋ₖ        bₓₖ = Im C₋ₖ − Im Cₖ
aᵧₖ = Im Cₖ + Im C₋ₖ        bᵧₖ = Re Cₖ − Re C₋ₖ
```

and the circle for frequency k in the complex form is cₖ = (aₓₖ + bᵧₖ)/2 + i(aᵧₖ − bₓₖ)/2.

### Choosing which circles to keep

In **Circles in circles** mode the app keeps the N circles with the **largest radii** |cₖ|, which gives the best picture for a given number of circles. In **X / Y guides** mode it keeps harmonics k = 1 … N on each axis.

### Lanczos σ-factors

Truncating a Fourier series makes it overshoot near sharp corners (the Gibbs phenomenon). Multiplying term k by

```
σₖ = sinc( k / (m + 1) ) = sin(π k /(m+1)) / (π k /(m+1))
```

where m is the highest frequency kept, smooths that overshoot at the cost of slightly softer corners.

---

## How a shape becomes circles (pipeline)

### 1. Outlines

**Text.** Each glyph is read from the font file by opentype.js as quadratic and cubic Bézier curves, which are flattened into dense points (under one unit apart at a 200-unit font size). No pixels are involved, which is why text stays sharp.

**Images.**
1. The image is drawn at the chosen detail resolution (1200 px by default, up to 2400 px).
2. Pixels are grouped into the chosen number of colours with **k-means++** clustering in the **CIELAB** colour space, so groups follow perceived colour rather than raw RGB.
3. A 3×3 majority filter removes speckles (adjustable).
4. The background colour (the colour touching the border most) is left out, if you choose.
5. For each colour, a **soft membership field** is computed: v = d_other / (d_this + d_other), using each pixel's CIELAB distance to its nearest and second-nearest colour. v = 0.5 exactly where the photo's own blended (anti-aliased) colour sits halfway between two colours, so edges land between pixels.
6. **Marching squares** follows every boundary of that colour; each edge point is placed where the field crosses 0.5 (sub-pixel accuracy). The tracer uses edge-indexed buffers allocated once, so large images do not exhaust memory.
7. Regions smaller than the chosen minimum area are dropped, and outlines are lightly smoothed.

**Draw.** Each stroke is a list of points. A stroke whose ends meet (within 20 % of its size) is a closed shape; otherwise it becomes an out-and-back loop with zero area, so it draws as a line and never fills.

**Equation.** The pasted text is parsed by a small, safe expression compiler (see below), each x(t), y(t) pair is sampled 1024 to 4096 times across the parameter range, and each curve becomes an outline. The y axis is flipped so equations from maths tools appear upright.

### 2. One path

All outlines are joined into one closed path. Text is visited in reading order; images and drawings use nearest-outline-next. Each outline is entered at the point closest to where the pen is, traced once around, and left by a pen-up travel move. Every sample remembers which outline it belongs to (or that it is travel), so the animation knows when to lift the pen and when each shape is complete.

### 3. Fourier transform

The path is resampled by arc length to **M = 2ⁿ** equal steps (2,048 up to 262,144, about two samples per unit of length) and transformed with an in-place **radix-2 Cooley–Tukey FFT**, O(M log M). Coefficients are sorted by radius, and a running "tail" sum of the radii left out is kept for the error bound.

### 4. Fitting to a quality level

For Draft, HD and Ultra HD, a **binary search** over the number of circles finds the smallest N whose widest gap to the outline is within the tolerance. Each test is a single **inverse FFT** with all but the chosen coefficients set to zero, so even a 262,144-sample path is checked in milliseconds.

### 5. The machine

At animation time t, the chain is summed term by term in double precision: c₀, then each circle's arm cₖ e^{ikt}, largest first. Every term is included, so the innermost arm ends exactly at the pen. Each arm gets its circle once that circle is large enough to see on screen.

---

## 3D and 4D

### 3D: x, y and z

Each outline is extruded into a solid with a front face at z = +d/2 and a back face at z = −d/2. One closed **space curve** traces each outline's front contour, an edge down, its back contour, then a pen-up move to the next outline. Each coordinate gets its own Fourier series:

```
x(t), y(t), z(t)  →  harmonic k = Aₖ cos kt + Bₖ sin kt,   with Aₖ, Bₖ ∈ ℝ³
```

so every harmonic traces an **ellipse in space** instead of a circle. This is the three-dimensional extension of elliptic Fourier descriptors. x and y share one complex FFT and z has its own:

```
Aₖ = ( Re Cₖ + Re C₋ₖ,  Im Cₖ + Im C₋ₖ,  Re Dₖ + Re D₋ₖ )
Bₖ = ( Im C₋ₖ − Im Cₖ,  Re Cₖ − Re C₋ₖ,  Im D₋ₖ − Im Dₖ )
```

Every outline is drawn as several laps: two for a flat prism, four for a **bevel** (front face inset, front edge, back edge, back face inset) and six for a **rounded** edge. The inset rings are offset along the outline's inward normal by a distance limited to 30 % of the shape's stroke thickness 2(A − holes)/P, so thin parts never fold. The single closed space curve visits every ring of every outline, so one machine still draws everything.

The finished solid is rendered with **WebGL 2**: walls are quad strips between matched rings (rings are matched by position along the outline, not by sample index), caps are triangulated with earcut, and a depth buffer, two lights, a specular highlight and MSAA replace the old painter's algorithm. The vertex shader applies exactly the same projection as the 2D overlay, so the ellipse chain and pen line up with the mesh. The WebGL canvas is copied into the main canvas, so PNG and video export still capture it. Without WebGL 2 (or with outline-only fill) the original canvas renderer is used.

### 4D: x, y, z and time

Time τ becomes the fourth coordinate. The coefficients themselves change with time:

```
r(t, τ) = Σₖ Σₙ c(k, n) · e^{i (k t + n τ)},      n ∈ {−1, 0, 1}
```

In practice the app blends the coefficients of the current shape A and a target B (a ring by default, or text you type) with s(τ) = (1 − cos τ)/2:

```
c(k, τ) = c_A(k) + ( c_B(k) − c_A(k) ) · s(τ)
```

which is exactly the double series above with c(k, 0) = (c_A + c_B)/2 and c(k, ±1) = (c_A − c_B)/4. The 3D object turns into the target and back once per period. One inverse FFT per frame gives the whole curve at the current moment, and faint copies show earlier moments (a "time trail").

**Shape sequences.** A cycle of shapes A → B → C → A is blended pairwise with s = (1 − cos πe)/2, where e is the eased position inside the step (with an adjustable pause on every shape). The time spectrum then has many more terms than n ∈ {−1, 0, 1}.

**Correspondence (why it no longer tangles).** Blending two Fourier series is only meaningful if equal parameter values mean equal places. Each target is therefore resampled on the first shape's clock: sample s of the target is the point at the same fraction of *visible* length as sample s of the first shape, where pen-up travel gets a small share (3 % in total) of the clock so a target never jumps. Its start point and running direction are chosen from 96 candidate phases × 2 directions by minimum distance. Each letter then morphs into its own arc of the target.

**Travelling wave.** z += h·sin(2π·n·x(t)/W − τ) is added in the time domain, and its own FFT is added to the circle chain, so the circles still end at the pen.

### Real depth from one photo (AI + 2D Fourier)

For uploaded photos, 3D can use real estimated depth instead of a flat extrusion:

1. **Depth estimation.** [Depth Anything V2 Small](https://github.com/DepthAnything/Depth-Anything-V2) (Apache-2.0) runs in the browser through [Transformers.js](https://github.com/huggingface/transformers.js) 3.8.1. It uses WebGPU when available and otherwise WebAssembly (8-bit model, about 27 MB, downloaded once and cached). The photo never leaves the device. The output is relative inverse depth, robustly normalised to 0…1 (1 = nearest) on a 256 × 256 grid.
2. **2D Fourier surface.** The grid is mirror-extended to 512 × 512, so there are no seams, then transformed with a 2D FFT (rows, then columns). Only harmonics with |m|, |n| ≤ K are kept, weighted by σ(m)·σ(n) Lanczos factors:
   `z(u,v) = Σ_{|m|≤K} Σ_{|n|≤K} σ(m) σ(n) d(m,n) e^{i(mu+nv)}`, giving (2K+1)² terms. K is the "Surface harmonics" slider.
3. **Lifting.** Each traced outline point (x, y) gets z = (½ − D(x, y))·h, where h is the relief height.
4. **Contours.** Iso-depth lines of the surface are traced with the same sub-pixel marching squares as the photo and placed at their constant depth.
5. **One machine.** Outlines and contours join with pen-up travel into one closed space curve. x(t), y(t) and z(t) get their own Fourier series, so every harmonic is an ellipse in ℝ³, drawn by the same ellipse chain as the other 3D modes.
6. **4D.** Time lifts the flat photo into its depth: r(t, τ) = r_flat(t) + s(τ)·(r_depth(t) − r_flat(t)), with s = (1 − cos τ)/2.

If the model cannot load (offline, or a blocked network), a clearly labelled approximation is used: smoothed brightness plus the "lower in the frame is nearer" cue.

### Verification

Both identities were checked numerically with the app's own FFT code: the 3D ellipse chain rebuilds a random space curve with a worst error of about 3 × 10⁻¹⁵, and the 4D blend matches the double Fourier series to about 1 × 10⁻¹⁶ (rounding error).

---

## Accuracy and the error guarantee

- The **widest gap** shown is the largest distance between the circles' curve and the true outline, measured at every pen-down sample, as a percentage of the drawing height.
- The **upper bound** is the sum of the radii left out. The truncated chain can never be further from the full series than that sum, because each left-out arm can move the pen by at most its own length (triangle inequality). With Lanczos damping on, this simple bound no longer applies and the readout says so.
- Reference values for the sample sentence at HD: 7,873 circles, widest gap 0.100 %. A 97-letter sentence needs about 9,700 circles at HD and about 35,800 at Ultra HD.

---

## Rendering and precision

- Finished shapes are drawn as smooth quadratic splines through the fitted samples, so curves stay smooth at any zoom.
- The view is anchored on the exact drawing point at the centre of the screen; screen positions are computed as (x − centre) × scale in double precision, so subtracting first avoids losing digits at extreme zoom.
- Beyond about 80,000 % zoom, fills, hatching and dashed outlines are drawn from coordinates the app computes itself (double precision), only for outlines in view, because the browser's canvas stores path coordinates in single precision.
- The circle chain is always summed in double precision, which is why the innermost circle (radius around 10⁻³ to 10⁻⁷ drawing units) can be shown clearly.
- Rendering uses up to 3× device pixel ratio for sharp lines on high-density screens.

---

## Technology

- **Plain HTML, CSS and JavaScript** in one file; no framework, no build step needed to run it.
- **Canvas 2D** for 2D drawing, the 4D curve and all overlays; **WebGL 2** for the 3D solid and the photo relief (earcut, 7 KB, is inlined for cap triangulation).
- **Transformers.js 3.8.1** with **Depth Anything V2 Small** (ONNX) for photo depth, loaded on demand.
- **opentype.js 1.3.4** (MIT) for reading font outlines, loaded from the jsDelivr CDN.
- **Embedded fonts** (WOFF, base64) for the five built-in typefaces, so text outlines work without network font access.
- **Google Fonts** for the interface typefaces (Bricolage Grotesque, Familjen Grotesk, DM Mono), with system fallbacks.
- **Web Audio API** for "Hear the shape".
- **MediaRecorder + canvas.captureStream** for video recording.
- Hand-written implementations of: radix-2 FFT, k-means++ in CIELAB, marching squares with sub-pixel interpolation, Ramer–Douglas–Peucker simplification, Catmull–Rom to cubic Bézier conversion, a recursive-descent maths parser, perspective projection, painter's sorting and Lambert shading.

---

## Project files

```
fourier-tracer.html      the complete app (built; fonts embedded); open it in a browser
src/app.html             the app source, with a __FONTS__ placeholder for the embedded fonts
build.py                 injects the base64 fonts into src/app.html to produce fourier-tracer.html
docs/README.md           this file
docs/INSTRUCTIONS.md     developer instructions: building, architecture, code map, extending, testing
docs/USER_MANUAL.md      every control explained, step-by-step workflows, tips and FAQ
```

The built page is about 345 KB, including about 140 KB of embedded fonts.

---

## Browser support

Current versions of Chrome, Edge, Firefox and Safari on desktop, Android and iOS. Notes:

- **HEIC** photos only open where the browser can decode them (mainly Safari).
- **WOFF2** fonts cannot be read by opentype.js; use TTF, OTF or WOFF.
- **Video recording** needs MediaRecorder with WebM support (Chrome, Edge, Firefox; Safari support varies).
- Inside some app views (for example an embedded preview in a mobile app), file pickers or downloads may be restricted by the host; opening the page in a normal browser avoids that.

---

## Privacy

Everything happens in your browser. Images, text, drawings and fonts you load are never uploaded. The page loads the opentype.js script and the interface fonts from public CDNs, and, only when you use real depth on a photo, Transformers.js (jsDelivr) and the depth model (Hugging Face). The model runs on your device.

---

## Known limits

- **Photos** cannot contain more detail than their pixels; tracing makes every edge smooth and vector-sharp, but fine texture becomes many small regions. Fewer colours and a larger minimum region give cleaner results.
- A thin line in a photo becomes a thin region with **two** outlines (one on each side), not a single stroke.
- **3D** for text, drawings and equations comes from extruding the flat drawing. For photos the AI depth is relative (right shape, not real distances), and portraits and other flat, front-on photos give only gentle relief.
- The AI depth model downloads about 27 MB on first use. Inside some hosts (for example the claude.ai preview) that download can be blocked; the app then uses its labelled approximate depth.
- **4D** morphs between two path structures in Fourier space, so in-between moments can look tangled; that is the true blend of the two series.
- Microscope, Compare and Machine-per-outline work in 2D only; turning one on switches back to 2D.
- Bevel and rounded edges need 2–3× the circles of a flat prism for the same accuracy. In 4D the number of harmonics is capped at 3,000, so very sharp targets (a star) show a little edge ripple.
- Interface copy protection stops casual selection and copying; it cannot stop someone reading the page source or using developer tools.
- Very long texts take up to two minutes to animate at 1×; use a higher speed or Finish.

---

## Methods and references

- J. W. Cooley and J. W. Tukey (1965). An algorithm for the machine calculation of complex Fourier series. *Mathematics of Computation*. (FFT)
- F. P. Kuhl and C. R. Giardina (1982). Elliptic Fourier features of a closed contour. *Computer Graphics and Image Processing*. (X / Y form, extended to 3D here)
- C. Lanczos (1956). *Applied Analysis*. (σ-factors)
- D. Arthur and S. Vassilvitskii (2007). k-means++: the advantages of careful seeding. (colour clustering)
- CIE (1976). CIELAB colour space.
- W. E. Lorensen and H. E. Cline (1987). Marching cubes. (marching squares is the 2D case)
- U. Ramer (1972); D. Douglas and T. Peucker (1973). Polyline simplification.
- E. Catmull and R. Rom (1974). A class of local interpolating splines.
- C. Brechbühler, G. Gerig and O. Kübler (1995). Parametrization of closed surfaces for 3-D shape description. (background on 3D shape description with harmonics)

---

## Credits and licences

- Fonts: **Archivo Black**, **Anton**, **Playfair Display**, **Rubik**, **Bricolage Grotesque**, all under the **SIL Open Font License 1.1**, embedded from the Fontsource packages.
- Interface fonts: **Bricolage Grotesque**, **Familjen Grotesk**, **DM Mono** via Google Fonts (SIL OFL 1.1).
- Font parsing: **opentype.js** (MIT licence).
- The example equations (butterfly curve by Temple H. Fay, heart curve, hypotrochoid, cardioid, nephroid, trefoil, Lissajous figure) are classical mathematical formulas; the rose and starfish equations are original to this project.
- Depth estimation: **Depth Anything V2 Small** (Apache-2.0) via **Transformers.js** (Apache-2.0).
- All application code, sample images and sample text are original to this project.

---

## Related documents

- **[INSTRUCTIONS.md](INSTRUCTIONS.md)**: how to build, run, deploy, read and extend the code.
- **[USER_MANUAL.md](USER_MANUAL.md)**: how to use every feature.
