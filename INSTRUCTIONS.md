# Fourier Outline Tracer — Developer Instructions

Everything you need to run, build, deploy, understand, change and test the project.
For what the app does and the mathematics behind it, see **[README.md](README.md)**. For how to use it, see **[USER_MANUAL.md](USER_MANUAL.md)**.

---

## Contents

1. [Quick start](#1-quick-start)
2. [Requirements](#2-requirements)
3. [Project layout](#3-project-layout)
4. [Building](#4-building)
5. [Running locally](#5-running-locally)
6. [Deploying](#6-deploying)
7. [Architecture overview](#7-architecture-overview)
8. [Code map (src/app.html)](#8-code-map-srcapphtml)
9. [State: the `S` object](#9-state-the-s-object)
10. [Data model](#10-data-model)
11. [Algorithms in detail](#11-algorithms-in-detail)
12. [Rendering pipeline](#12-rendering-pipeline)
13. [Input and camera](#13-input-and-camera)
14. [Tuning constants](#14-tuning-constants)
15. [Extending the app](#15-extending-the-app)
16. [Testing](#16-testing)
17. [Performance notes](#17-performance-notes)
18. [Security and safety](#18-security-and-safety)
19. [Troubleshooting](#19-troubleshooting)
20. [Coding conventions](#20-coding-conventions)
21. [Licences](#21-licences)

---

## 1. Quick start

```bash
# build the single-file app (embeds the fonts)
python3 build.py

# open it
open fourier-tracer.html            # macOS
xdg-open fourier-tracer.html        # Linux
start fourier-tracer.html           # Windows
```

No package install, bundler or server is needed. The page needs internet access once to load opentype.js and the interface fonts from public CDNs.

---

## 2. Requirements

| Purpose | Requirement |
|---|---|
| Run the app | Any current browser (Chrome, Edge, Firefox, Safari; desktop or mobile) |
| Build | Python 3.8 or newer (standard library only) |
| Automated tests (optional) | Python 3 with `playwright` and Chromium (`pip install playwright && playwright install chromium`), plus `Pillow` for pixel checks |
| Syntax check (optional) | Node.js (`node --check`) |

---

## 3. Project layout

```
.
├── fourier-tracer.html     built app: one self-contained page (≈300 KB)
├── build.py                build script: embeds fonts into src/app.html
├── src/
│   └── app.html            the source: markup, CSS and JavaScript, with a __FONTS__ placeholder
├── fonts/                  the five embedded typefaces (WOFF, SIL OFL 1.1)
│   ├── archivo-black-latin-400-normal.woff
│   ├── anton-latin-400-normal.woff
│   ├── playfair-display-latin-900-normal.woff
│   ├── rubik-latin-900-normal.woff
│   └── bricolage-grotesque-latin-800-normal.woff
└── docs/
    ├── README.md
    ├── INSTRUCTIONS.md     (this file)
    └── USER_MANUAL.md
```

Always edit **src/app.html**, then run `build.py`. Never hand-edit `fourier-tracer.html`; it is overwritten on every build.

---

## 4. Building

`build.py`:

1. Reads `src/app.html` and checks it contains the literal `__FONTS__`.
2. Reads the five WOFF files listed in its `FONTS` table from `fonts/`.
3. Base64-encodes each and replaces `__FONTS__` with a JSON array `[{ "name": …, "data": … }, …]`.
4. Writes `fourier-tracer.html`.

At runtime, `EMBEDDED_FONTS` holds that array; `initFonts()` decodes each entry with `opentype.parse()` and fills the font menu.

The fonts are the Latin subsets from the Fontsource packages (`@fontsource/archivo-black`, `@fontsource/anton`, `@fontsource/playfair-display`, `@fontsource/rubik`, `@fontsource/bricolage-grotesque`, version 5.3.0). Latin covers U+0000–00FF, which includes German umlauts and ß.

---

## 5. Running locally

Open `fourier-tracer.html` directly from disk (`file://`) or through any static server:

```bash
python3 -m http.server 8000      # then visit http://localhost:8000/fourier-tracer.html
```

When the page is opened directly (not inside the claude.ai artifact viewer), **Save** buttons use a normal browser download. Inside the artifact viewer they use the viewer's `downloads` capability (see §6).

---

## 6. Deploying

The app is static. Any of these work: a plain web server, GitHub Pages, Netlify, Vercel, S3 or Cloudflare Pages, or a claude.ai artifact.

### External resources the page loads

| Resource | Host | Needed for |
|---|---|---|
| `opentype.js@1.3.4/dist/opentype.min.js` | cdn.jsdelivr.net | Reading font outlines (text source, custom fonts, 4D text target) |
| Interface fonts (Bricolage Grotesque, Familjen Grotesk, DM Mono) | fonts.googleapis.com / fonts.gstatic.com | Interface typography only; system fonts are used if blocked |
| `@huggingface/transformers@3.8.1` (ES module, dynamic `import()`) and its ONNX Runtime WASM files | cdn.jsdelivr.net | Photo depth (3D/4D "Real depth (AI)"), loaded only when used |
| `onnx-community/depth-anything-v2-small` (`model_quantized.onnx`, about 27 MB, or `model_fp16.onnx` on WebGPU) | huggingface.co (and its CDN redirects) | Photo depth; cached by the browser after the first download |

To make the page fully offline, inline opentype.js into `src/app.html` (replace the `<script src=…>` tag with the library's contents) and drop the Google Fonts link (the CSS already has fallbacks).

If opentype.js fails to load, the text source falls back to rendering text onto a canvas and tracing it as an image (`buildTextRaster`).

If Transformers.js or the depth model cannot load, `estimateDepth` falls back to a labelled approximate depth (smoothed brightness plus "lower in the frame is nearer"), and the note under the 3D settings says so.

### Content Security Policy

If you serve it with a CSP, allow:

```
script-src 'self' 'unsafe-inline' 'wasm-unsafe-eval' https://cdn.jsdelivr.net;
connect-src 'self' https://cdn.jsdelivr.net https://huggingface.co https://*.huggingface.co https://*.hf.co;
worker-src blob:;
style-src  'self' 'unsafe-inline' https://fonts.googleapis.com;
font-src   https://fonts.gstatic.com data:;
img-src    'self' data: blob:;
media-src  blob:;
```

The equation compiler uses `new Function(...)`, so `'unsafe-eval'` is also required for the Equation source. See §18 for why this is safe. `connect-src`, `worker-src` and `'wasm-unsafe-eval'` are only needed for photo depth; without them that feature falls back to the approximate depth.

### claude.ai artifact

The artifact declares the `downloads` capability so Save SVG / PNG / video work inside the viewer: `claude.use('downloads')` → `downloads.save({ filename, data })`. Outside the viewer, `window.claude` is undefined and a normal `<a download>` link is used instead.

Inside the viewer, the depth model download and the Fullscreen API may be blocked by the host. Both degrade gracefully: approximate depth, and the full-screen button and prompt stay hidden.

---

## 7. Architecture overview

One IIFE in `src/app.html` holds the whole app. The data flow:

```
 source (text | image | draw | equation)
        │
        ▼
 outlines  ── prepLoop() per outline: smoothed raw points, resampled samples, small per-outline Fourier series
        │
        ▼
 finalizeModel(): computeGroups (outer shape + its holes), buildGlobal() → one closed path → FFT
        │                         (3D: buildS3 → space curve, 4D: build4D → two space curves)
        ▼
 applyN(): fit to the quality level (binary search with inverse FFTs) → fitted samples → Path2D per outline
        │
        ▼
 animation loop (frame): advance time → draw(): fills, dashed targets, circle chain, pen, overlays
```

Key ideas:

- **One global path** (`model.G`) carries every outline plus pen-up travel. Each sample knows its owner outline (`own[j]`, or −1 for travel), and each outline knows its sample range (`slice[i]`). This lets one series draw everything while the app still knows when each shape is finished.
- **Two kinds of Fourier unit** share one interface: the global unit `G` (FFT-based, `isG: true`) and per-outline units (direct DFT, at most 200 harmonics) used by *Machine per outline*. Functions such as `recon`, `maxTerms`, `bound`, `fitUnit`, `fullArms` and `chainAt` accept either.
- **Rendering is layered**: a cached `done` canvas for finished fills, a cached `ghost` canvas for dashed targets, and the live canvas for grid, chain, pen and overlays, so a frame does not repaint everything.

---

## 8. Code map (src/app.html)

Sections appear in this order (search for the `/* ---------- name ---------- */` banners):

| Section | Main functions | Purpose |
|---|---|---|
| utils | `rng`, `trig`, `toast`, `hex`, `sinc`, `nextPow2`, `busy` | Seeded RNG, cached cos/sin tables, helpers |
| radix-2 FFT | `fftTables`, `fft(re, im, inverse)` | In-place Cooley–Tukey FFT with cached bit-reversal and twiddle tables |
| colour quantisation | `toLab`, `kmeans`, `modeFilter` | sRGB→CIELAB, k-means++ (14 iterations, seeded), nearest and second-nearest cluster per pixel, 3×3 majority filter |
| marching squares | `fieldFor`, `traceLabel`, `area`, `smoothLoop`, `perimeter`, `resampleClosed` | Soft colour field, sub-pixel contour tracing with reusable edge-indexed buffers |
| vector text | `initFonts`, `flattenCommands`, `wrapLines`, `textContours`, `buildText`, `buildTextRaster` | Font loading, Bézier flattening, line wrapping, glyph outlines with per-letter colour |
| per-outline units | `realDFT`, `prepLoop` | Direct DFT up to 200 harmonics; per-outline circle terms sorted by radius |
| global path | `buildGlobal`, `fk`, `maxTerms`, `errorOf`, `reconG`, `reconLoop`, `recon`, `bound`, `fitUnit` | Join outlines, resample to 2ⁿ, FFT, reconstruction by inverse FFT, error and bound, quality fit |
| model | `processImage`, `computeGroups`, `finalizeModel`, `loopPts`, `pathFrom`, `tolAbs`, `applyN` | Build the model from any source; outer/hole grouping; fit and readouts |
| animation | `seq`, `unit`, `resetAnim`, `finish`, `progress`, `seek`, `advance` | Timeline, seeking, hatch reveal, time dilation, auto-stop of recordings |
| view | `layout`, `applyView`, `resetView`, `zoomAt`, `panBy`, `SX`, `SY`, `zoomLabel`, pointer and wheel handlers | Centre-anchored camera, fit, zoom and pan, compare panes, follow camera |
| rendering | `bboxOf`, `viewRect`, `screenPath`, `deepPathOf`, `clipSeg`, `paintGhost`, `hatchSegments`, `fillLayers`, `paintDone`, `drawGrid`, `armsOf`, `penAt`, `circleChain`, `fullOrth`, `orthAt`, `clipLine`, `guideChains`, `drawTrace`, `directionArrow`, `draw`, `tag`, `updateHud`, `frame` | Everything drawn in 2D |
| microscope | `fullArms`, `chainAt`, `microCamera`, `drawMicro`, `overlay`, `setMicro`, `setDepth` | Exact full chain, depth camera, time dilation readout |
| convergence sweep | `startSweep`, `stopSweep`, `stepSweep` | Grow N over 12 s |
| hear the shape | `startAudio`, `stopAudio` | Stereo looping buffer from the fitted path |
| record | `toggleRecord` | MediaRecorder on `canvas.captureStream(60)` |
| draw your own | `drawEditor`, `strokeLen`, `isClosed`, `eraseAt`, `setTool`, `buildDrawing`, `refreshDrawUI` | Sketch editor, eraser, open/closed stroke handling; the playback bar's play / restart / finish call `buildDrawing` while editing |
| photo depth | `depthPipe`, `normDepth`, `estimateDepth`, `fourierSurface`, `depthAt`, `reliefH`, `zOf`, `depthContours`, `contourCol`, `reliefCurve`, `ownerCol`, `drawDepthPreview`, `applyDepthK`, `depthWanted`, `ensureDepth`, `drawRelief` | Depth Anything V2 in the browser, 2D Fourier depth surface, iso-depth contours, relief space curve and its renderer |
| 3D and 4D | `profileFor`, `insetNormals`, `spaceCurve`, `reconS`, `errS`, `ellipsesOf`, `buildS3`, `fit3D`, `setP3`, `pj`, `shade`, `floorGrid`, `chain3`, `draw3D`, `polyPts`, `UNIT_SHAPES`, `morphShape`, `morphTarget`, `visCum`, `remapTo`, `build4D`, `draw4D`, `setDim`, `stopAutoOrbit` | Space curves with bevel rings, 3D ellipse chain, space-time morph with correspondence |
| WebGL | `glInit`, `glBegin`, `glUpload`, `ringPoints`, `wallMesh`, `capMeshes`, `drawSolidGL`, `reliefMesh`, `drawReliefGL` | WebGL 2 renderer for the solid and the photo relief (shaders `GL_VS`, `GL_FS`) |
| 3D / 4D export | `exportMeshes`, `toGltf`, `buildGLB`, `buildSTL`, `saveGLB`, `saveSTL`, `gifPalette`, `buildGIF`, `saveGif4D` | GLB, STL and looping GIF writers (no libraries) |
| spectrum | `drawSpec` | Radius chart (log scale) or x/y harmonic bars |
| export | `rdpClosed`, `bezierD`, `buildSvg`, `offer`, `saveSvg`, `savePng`, `copyText`, `copySvg`, `copyJson` | SVG with cubic Béziers, PNG 4K, JSON coefficients |
| equation export | `nf`, `eqCurves`, `eqText`, `scheduleEq` | Formula text in five formats |
| equation import | `cleanMath`, `splitIdent`, `compileExpr`, `splitTop`, `matchClose`, `extractCurves`, `EXAMPLES`, `setFill`, `simulateEquation`, `setCompare` | Safe expression compiler and format detection |
| sources | `setK`, `picCanvas`, `sampleCanvas`, `drawSample`, `loadFile`, `loadFont`, `setSource` | Ready-made drawings (painted on a canvas, traced, turned into editable strokes), uploads, fonts, tab switching |
| controls | `refreshUI`, `setMode`, `setMachine`, `setQuality`, `setSpeed`, `setBg`, `bind`, all event wiring, equation auto-simulate (`eqAuto`), phone HUD (`syncMobPlay`, `syncMobSpeed`, `setSpop`), slide-in menu (`setMenu`, `drawerMode`), full screen (`goFull`), copy protection, boot | UI wiring and start-up |

---

## 9. State: the `S` object

All user settings live in one object `S`:

| Key | Default | Meaning |
|---|---|---|
| `source` | `'txt'` | Active source: `'txt'`, `'img'`, `'draw'`, `'eq'` |
| `mode` | `'circles'` | `'circles'` (complex form) or `'orth'` (X / Y guides) |
| `machine` | `'single'` | `'single'` (one fixed machine) or `'each'` (per outline) |
| `quality` | `'hd'` | `'draft'`, `'hd'`, `'ultra'`, `'manual'` |
| `N` | 2000 | Number of terms in Manual quality |
| `lanczos` | false | Apply σ-factors |
| `k`, `res`, `minArea`, `smooth`, `clean`, `dropBg` | 2, 1200, 16, 1, 1, true | Image tracing: colours, resolution, minimum region, smoothing radius, speckle passes, drop background |
| `wrap` | 22 | Text wrap width in characters (0 = off) |
| `fillStyle` | `'solid'` | `'solid'`, `'lines'`, `'none'` |
| `hatchN`, `hatchW`, `hatchA`, `cross`, `hatchP` | 60, 1.2, 45, false, 1 | Hatch line count, thickness (px at Fit), angle, cross-hatch, reveal progress |
| `ghost`, `arrow`, `grid`, `neon` | true, true, true, false | Display toggles |
| `bg` | `''` | Canvas background (empty = follow theme) |
| `arms` | 60 | Circles drawn at normal zoom (more appear when large enough on screen) |
| `speed` | 1 | Playback speed, 0.03–16 |
| `playing` | true | Playing or paused |
| `compare` | false | Show the original photo side by side |
| `micro`, `depth`, `diving`, `diveDir` | false, 0.3, false, 1 | Microscope state |
| `follow` | false | Camera follows the last circle |
| `followAxis` | `'x'` | In X / Y mode, which machine's last circle the camera follows |
| `pitch` | 110 | Laps per second for sound |
| `editing`, `tool` | true, `'pen'` | Draw-mode editor state |
| `eqFmt`, `eqScope`, `eqTerms` | `'app'`, `'outline'`, 40 | Equation export settings |
| `dim`, `depth3`, `morph`, `tauSpeed`, `trail4` | `'2d'`, 40, `'ring'`, 0.4, true | 3D / 4D settings (`morph` is `'ring'`, `'text'` or `'seq'`) |
| `shape`, `bevel` | `'bevel'`, 60 | 3D edge profile (`'flat'`, `'bevel'`, `'round'`) and edge size in % |
| `dwell`, `waveAmp`, `waveN` | 20, 0, 3 | 4D pause on each shape (%), travelling wave height (%) and number of waves |
| `depthTex` | true | Paint the photo on the depth surface (WebGL) |
| `zmode`, `depthK`, `depthLines`, `depthMesh` | `'ai'`, 24, 10, true | Photo depth: AI relief or extrusion, surface harmonics K, contour lines, show the surface mesh |
| `svgBg` | false | Include background in SVG and PNG |

---

## 10. Data model

`model` (built by `finalizeModel`):

| Field | Meaning |
|---|---|
| `loops[]` | Outlines. Each has `raw` (smoothed points), `orig` (resampled), `M`, `K`, `ax/bx/ay/by`, `terms` (sorted circles), `tail`, `c0`, `h`, `label` (colour index), `per`, `area`, `opath` (Path2D of `raw`), `path` (Path2D of the fitted curve), `n` (terms used), `open` (draw mode, open stroke) |
| `layerOrder[]` | Colour indices in painting order |
| `cols[]` | Hex colour per label |
| `W`, `H` | Drawing size in drawing units |
| `ow`, `oh` | Export size (original image pixels, or drawing units) |
| `rule` | Fill rule: `'evenodd'` (images, drawings, equations) or `'nonzero'` (text) |
| `kind` | `'text'`, `'image'`, `'draw'`, `'equation'` |
| `srcImg` | Original image (compare view) |
| `group[]`, `members` | Outer outline that each hole belongs to; fills wait until a whole group is traced |
| `G` | The global unit (below) |
| `S3`, `S4` | 3D space curve and 4D morph data (only when in 3D / 4D) |

`model.G` (built by `buildGlobal`):

| Field | Meaning |
|---|---|
| `M` | Sample count, a power of two (2,048 to 262,144) |
| `orig` | Resampled path, interleaved x, y |
| `own` | Owner outline per sample, −1 for travel |
| `slice[i]` | `[first, last]` sample index of outline i |
| `order` | Outline visiting order |
| `cRe`, `cIm`, `mag` | FFT coefficients (scaled by 1/M) and their magnitudes |
| `sorted` | Coefficient indices (excluding 0) by descending magnitude |
| `tail` | `tail[n]` = sum of magnitudes from rank n onwards (error bound) |
| `c0` | Centre (DC term) |
| `h` | Height of the pen-down part, for percentages |
| `n`, `rec`, `err` | Terms in use, fitted samples, error report |
| `full` | Cached arrays of every term in use (`K`, `RE`, `IM`, `R`, chain positions `PX`, `PY`) |

Frequency of index j: `fk(j, M) = j < M/2 ? j : j − M`.

---

## 11. Algorithms in detail

### 11.1 FFT
`fft(re, im, inverse)` is an iterative radix-2 Cooley–Tukey transform. Bit-reversal and twiddle tables are cached per size. Forward uses e^{−iθ}; inverse uses e^{+iθ} and **does not** divide by M (coefficients are pre-divided after the forward pass).

### 11.2 Fitting
`fitUnit(U, tol)`: compute the full-detail error; if it meets the tolerance, binary-search N between 1 and the maximum. Each probe is `recon(U, N)`, an inverse FFT with all but the chosen coefficients zeroed. The tolerance is `TOL[quality] / 100 × G.h`.

### 11.3 Reconstruction choices
- Circles mode keeps the N largest coefficients (`sorted[0 … N−1]`).
- X / Y mode keeps |k| ≤ N.
- Lanczos weights use m = the highest |k| kept.

### 11.4 Image tracing
- `kmeans`: samples about 12,000 pixels, seeds with k-means++, runs 14 Lloyd iterations in CIELAB, then assigns every pixel its nearest **and** second-nearest cluster with distances `d1`, `d2`.
- `fieldFor(L)`: v = d_other / (d_L + d_other) (0.75 / 0.25 for pixels the majority filter flipped), then a 3×3 binomial blur.
- `traceLabel`: marching squares over the binary label mask (topology) with crossing points where v = 0.5 (position, clamped to 3–97 % along the edge). Saddle cases are split. Buffers are edge-indexed (`2·(W+2)·(H+2)`), allocated once and reset only where used.

### 11.5 Outer and hole grouping
`computeGroups`: for each outline, the smallest same-colour outline of larger area that contains its first point is its parent. Fills appear only when an outline's whole group is finished; until then traced parts are drawn as strokes. This avoids a hole showing filled before it is cut out.

### 11.6 Joining outlines
`buildGlobal`: text uses reading order; images use nearest-next from the current pen position (scanning every 16th point, then refining). Each outline is entered at its nearest point, traced once and closed; travel segments are owner −1. The total length sets M ≈ 2 samples per unit (rounded up to a power of two, clamped).

### 11.7 Export curves
`bezierD`: Ramer–Douglas–Peucker simplification (ε = 0.025 % of drawing height, minimum 0.02 units) on the closed polyline, then a closed Catmull–Rom spline converted to cubic Béziers: control points p₁ + (p₂ − p₀)/6 and p₂ − (p₃ − p₁)/6.

### 11.8 Equation compiler
`compileExpr(src)`:
1. `cleanMath` normalises LaTeX (`\left`, `\right`, `\cdot`, `\frac{a}{b}`, `\sqrt{…}`, `\sin` …), brackets, `**`, π, θ, `np.` / `Math.` prefixes and Unicode minus signs.
2. A tokenizer accepts only numbers, identifiers and single-character operators. Identifiers are split into known names (`t`, `theta`, `pi`, `e`, and functions: sin, cos, tan, exp, sqrt, abs, log/ln, asin, acos, atan, sinh, cosh, tanh, floor, ceil, sign, sec, csc, cot, arcsin, arccos, arctan); anything else is an error.
3. A recursive-descent parser handles + − × ÷ ^, unary signs, implicit multiplication (`3t`, `2 sin(t)`), and powers on functions (`sin^2 t`).
4. The JavaScript it emits contains only numbers, `t`, `Math.*` calls from a fixed table, operators and parentheses, and is compiled with `new Function('t', …)`.

`extractCurves(text)` detects the format:

| Format | Detection |
|---|---|
| GeoGebra | `Curve(x, y, t, a, b)` (comment lines ignored) |
| Mathematica | `ParametricPlot[{{x1, y1}, …}, {t, a, b}]`, `RGBColor[r, g, b]` colours |
| App / generic | `x(t) = …` followed by `y(t) = …`; `# outline N colour #rrggbb`, `# t from a to b` comments |
| Desmos | `\left(x, y\right)` lines, domain 0 to 1 |
| Python | `("#rrggbb", x, y)` tuples, `linspace(a, b, …)` domain |
| Pair | any `(x, y)` line |

### 11.8b 3D equations
In 3D/4D, `eqCurves` adds a z series to each curve (`zSeries`: real DFT of the outline's z along the same resampled points `L.orig`, flipped so +z points at the viewer); the whole-drawing scope converts the ellipse chain `C.ell` to three real cosine series. `eqText` writes `z(t)` lines (App), `(x, y, z)` triples (Desmos 3D), `Curve(x, y, z, t, 0, 2pi)` (GeoGebra), `ParametricPlot3D` (Mathematica) or a Matplotlib 3D plot (Python), with a `# 3D · extruded solid, depth N%` or `# 3D · real depth relief` header. `extractCurves` reads all five 3D forms and the header (`solid`). `simulateEquation` then either sets `depth3` and switches to 3D (solid), or stores per-point z on each loop (`L.z`, model flag `z3`), which `buildS3` → `reliefCurve` uses directly; `drawRelief` renders it. Pasting coloured outlines restores the fill style that a plain example switched off (`S.fillPrev`).

### 11.9 3D space curve
`spaceCurve(loops, order, depth, M)`: front contour at +d/2, edge down, back contour at −d/2, travel. x and y share one complex FFT; z has its own. `ellipsesOf` builds Aₖ, Bₖ ∈ ℝ³ per harmonic (formulas in README). `fit3D` binary-searches K by 3D distance.

### 11.10 Photo depth relief
`ensureDepth` runs Depth Anything V2 once per photo (WebGPU fp16, else WASM q8) and stores a normalised 256 × 256 grid. `fourierSurface` mirror-extends it to 512 × 512, applies a 2D FFT, keeps |m|, |n| ≤ K with σ(m)σ(n) weights and inverts it. `reliefCurve` lifts every outline onto the surface (z = (½ − D)·h, h = 1.4 × depth3 % × drawing height), appends iso-depth contours from `depthContours` (marching squares on the surface), joins everything with pen-up travel and runs the same per-axis FFT as `spaceCurve`. Owners ≥ `model.loops.length` are contours. In 4D the target is the same curve with z = 0, so time lifts the flat photo into its depth.

### 11.11 4D morph
`build4D`: the current shape and every target are turned into space curves with the same M (16,384). Each target is then re-timed by `remapTo` onto the first shape's clock (equal fraction of visible length, travel gets 3 % of the clock; start phase and direction chosen by minimum distance over 96 × 2 candidates) and its spectrum is recomputed with one FFT. `draw4D` picks the pair (i, i+1) from τ, eases s, blends coefficients, optionally adds the travelling-wave spectrum, runs two inverse FFTs, and draws the curve (broken whenever the owner outline changes), the time trail (snapshots every 0.22 rad of τ, up to 9) and the blended ellipse chain. No arrays are allocated per harmonic in the frame loop.

### 11.12 Solid profile and WebGL
`profileFor` returns the rings `[inset, z]` (flat 2, bevel 4, round 6). `spaceCurve` makes one lap per ring; every sample gets `O` (outline), `Pt` (ring index, or `RING_EDGE` for the step between rings) and `Q` (fractional raw-point index, the position along the outline). `ringPoints` takes N samples of ring 0 and finds the sample with the same `Q` in every other ring (strictly increasing, so no duplicate points); `wallMesh` builds quad strips with smooth normals along the outline (and across rings for the rounded profile); `capMeshes` triangulates each nesting group (outer outlines plus holes, from `m.parentOf` and `L.hole`) with earcut. `L.thick` (stroke thickness of the group) limits the inset to 30 % of it. Meshes are cached per outline in `GL.cache` and rebuilt when the fit, depth, edge or Lanczos setting changes. Caps of a group appear once the whole group has been traced.

---

## 12. Rendering pipeline

Per frame (`frame` → `draw`):

1. **Editor** (Draw source, editing): `drawEditor` only.
2. Background and grid.
3. **Microscope** → `drawMicro`; **3D** → `draw3D` (or `drawRelief` for a photo with depth); **4D** → `draw4D` (each returns early).
4. **Compare**: draw the original image in the left pane with the same transform, then clip to the right pane.
5. **Fills** from the cached `done` canvas, repainted at most every 120 ms during animation or immediately when the view changes. Beyond `DEEP` zoom, `deepPathOf` builds screen-space paths in double precision for visible outlines only.
6. **Dashed targets** from the cached `ghost` canvas, drawn on top of the fills for unfinished shapes.
7. **Machine**: `circleChain` (full chain, culled, every arm with a visible circle) or `guideChains` (both X / Y machines with every harmonic, double precision, lines clipped by `clipLine`); then `drawTrace` (current outline in progress, pen, direction arrow).
8. **Overlays**: compare tags, follow readout, convergence readout.

Line widths in image space are divided by the scale so strokes stay a constant number of screen pixels.

---

## 13. Input and camera

- The 2D camera is anchored on `(view.ux, view.uy)`, the drawing point at the centre of the drawing pane, with scale `view.s = view.bs × view.z` (`bs` = fit scale). Screen x = `(x − ux) × s + scx`. `view.ox`, `view.oy` are derived for code that uses canvas transforms at moderate zoom.
- `zoomAt(mx, my, f)` keeps the point under the cursor fixed (or keeps the followed circle centred). `panBy(dx, dy)` moves the anchor. `ZMAX = 1e14`.
- **Follow camera**: `stepFollow` recentres on the last circle every frame (`lastCircleAt`: the complex chain, or the chosen X / Y machine) and runs the fly-in zoom (6.5 s exponential).
- **Microscope**: wheel, pinch and + / − change depth instead of zoom.
- **3D / 4D**: drag turns the object the way the hand moves (`yaw -= dx·k`, `pitch += dy·k`, half a turn across the canvas): drag right turns it right, drag down tilts it down. Shift-drag, right-drag or middle-drag pans; wheel or pinch zooms; two-finger drag pans; dragging stops auto-orbit.
- **Touch**: pointer events with `touch-action: none`; two quick taps (under 320 ms, within 36 px) zoom in, because browsers do not reliably send `dblclick` for touch; `setPointerCapture` is wrapped in try/catch.
- **Draw editor**: pointer events draw or erase strokes in canvas pixels.
- Keyboard on the focused canvas: `+` / `=` zoom in, `-` zoom out, `0` fit.

---

## 14. Tuning constants

| Constant | Value | Effect |
|---|---|---|
| `TOL` | draft 0.5, hd 0.1, ultra 0.025 | Allowed gap in % of drawing height |
| `MAX_LOOPS` | 1500 | Most outlines kept from an image (largest first) |
| `KCAP_LOOP` | 200 | Harmonics per outline in per-outline mode |
| `ARM_CAP` | 900 | Arms kept by `armsOf` (spectrum, legacy); the X / Y chains on screen use every harmonic |
| `M_MIN`, `M_MAX` | 2,048, 262,144 | Sample range of the global path |
| `DEEP` | 800 | Zoom factor above which deep (double-precision) fill rendering is used |
| `ZMAX` | 1e14 | Maximum canvas zoom factor |
| 4D `M` | 16,384 | Samples per space curve in 4D |
| 4D `K` cap | 3,000 | Harmonics per space curve in 4D |
| Sweep duration | 12 s | Convergence sweep length |
| Fly-in duration | 6.5 s | Zoom into the last circle |
| `DEPTH_N` | 256 | Depth grid size (512 × 512 after mirroring) |
| Depth contours | ≤ 240 loops | Cap on iso-depth contour loops |
| Phone image resolution | 800 px | Default detail resolution on touch screens up to 900 px |
| Equation auto-simulate delay | 700 ms | After the last keystroke |

---

## 15. Extending the app

### Add a built-in font
1. Put the WOFF (not WOFF2) file in `fonts/`.
2. Add `("Display name", "file.woff")` to `FONTS` in `build.py`.
3. Run `python3 build.py`. The menu is filled automatically.

### Add an equation export format
1. Add an `<option>` to `#eqFmt`.
2. In `eqText(fmt)`, add entries to the `ang`, `fn`, `mul` tables and a branch that assembles the curves.
3. If it should also import, teach `extractCurves` to recognise it.

### Add a fill style
Add a button with `data-fill="name"` and a branch in `fillLayers` (and in `buildSvg` for export).

### Add a ready-made drawing
Add a branch in `picCanvas(kind)` that paints a 960 × 720 canvas and returns `{ c, k }` (k = colours), and a button with `data-dsample="kind"` in the Draw panel. `drawSample` traces it and turns the outlines into editable strokes.

### Add an example equation
Add an entry to `EXAMPLES` (comments may set `# t from a to b` and `# colour #rrggbb` per curve) and a button with `data-ex="key"`.

### Add a morph target (4D)
Add a button with `data-morph="name"` and return `{ loops, cols }` for it in `morphTarget`.

### Change quality presets
Edit `TOL`. Values are percentages of the drawing height.

---

## 16. Testing

The project was verified with headless Chromium through Playwright. Useful checks:

```python
# load the built page with opentype.js served locally (so tests run offline)
OT = open('opentype.min.js').read()
await page.route('**/opentype.min.js', lambda r: r.fulfill(status=200, content_type='application/javascript', body=OT))
await page.set_content('<!doctype html><html><head><meta charset=utf-8><meta name=viewport content="width=device-width,initial-scale=1"></head><body style="margin:0">' + open('fourier-tracer.html').read() + '</body></html>')
```

Recommended test list:

| Area | Check |
|---|---|
| Load | No `pageerror`; status line shows glyph and outline counts |
| Text | Long sentence builds; HD reports a gap of 0.100 % |
| Image | Upload a photo with `set_input_files('#file', …)`; outline count reported |
| Equation round trip | For each `#eqFmt` option, copy `#eqOut` into `#eqIn` (or type with `page.type`) and wait about 1 s for auto-simulate; the curve count matches the outline count |
| Examples | Each `[data-ex]` button simulates without error |
| Playback | `#scrub`, `#backBtn`, `#fwdBtn` change `#progVal` |
| Zoom | Repeated wheel events reach the limit; Fit returns to 100 % |
| Follow | `#lastBtn`, wait 7 s; overlay shows "Following the last circle" |
| 3D / 4D | `[data-dim]` buttons; frame rate stays near 60 fps |
| Draw | Draw strokes with `mouse.down/move/up`; eraser splits; `#playBtn` builds outlines; each `[data-dsample]` loads strokes |
| Layout | At 1280×800, 375×812 and 812×375 (with `is_mobile`, `has_touch`), no element extends beyond the viewport and `scrollWidth ≤ clientWidth`; on phones the playback bar is one row and `#menuBtn` opens and closes the drawer |
| Touch | Synthetic `PointerEvent`s with `pointerType: 'touch'`: double-tap doubles the zoom; a two-finger parallel drag in 3D pans without zooming |
| WebGL 3D | Text in 3D with Flat, Bevel and Rounded edges: no `pageerror`, overlay says WebGL; outline-only fill falls back to the canvas renderer without errors |
| 4D sequence | Morph text `star \| heart`: at τ = 0 the shape equals the 3D object, at the end of the first step it is the star, with no ringing between |
| Exports | GLB: header, JSON, accessor counts, indices in range (Pillow or trimesh can load it); STL: triangle count matches file size; GIF: Pillow reports 48 frames and loop 0 |
| Photo depth | Launch Chromium with the session proxy; upload an image, switch to 3D; `#depthNote` reports Depth Anything V2. With jsDelivr / Hugging Face blocked by `page.route`, it reports the approximate fallback without errors |
| 3D drag direction | Drag right: the left end of the text comes towards the viewer; drag down: the tops of the letters become visible |
| Copy protection | Selecting interface text yields an empty selection; text fields still select and type |

A numerical check of the 3D/4D identities can reuse the app's FFT: extract the FFT section from `src/app.html` into a script, build a random space curve from known harmonics, transform it, rebuild it with the ellipse formulas, and confirm the error is around 10⁻¹⁵.

Syntax check after edits:

```bash
python3 -c "import re;s=open('fourier-tracer.html').read();open('/tmp/app.js','w').write(re.findall(r'<script>(.*?)</script>',s,re.S)[-1])"
node --check /tmp/app.js
```

---

## 17. Performance notes

- FFT size dominates: a 262,144-sample path takes a few milliseconds per transform; a fit runs about 18 inverse transforms.
- `chainAt` sums every term each frame (tens of thousands of cos/sin pairs); arms smaller than 0.05 px are not drawn.
- Fill repaints are throttled to 120 ms while drawing and are immediate on zoom or pan.
- Deep zoom builds screen paths only for outlines whose bounding box touches the view.
- Image tracing time grows with resolution × colours; 1200 px with 6 colours takes about one second on a laptop.
- Measured: 55–61 fps in 2D, 3D and 4D on a laptop-class machine with the sample sentence at HD.

---

## 18. Security and safety

- **No uploads.** All processing is local; no data leaves the browser.
- **Equation compiler.** Pasted text is never evaluated directly. The tokenizer rejects any identifier that is not a known maths name, and the emitted code is built only from numbers, `t`, whitelisted `Math` functions, operators and parentheses. It runs in strict mode with no access to page variables.
- **Copy protection.** CSS `user-select: none` on the interface, plus capture-phase handlers that cancel `copy`, `cut`, `selectstart`, `dragstart` and context menus outside text fields. The app's Copy buttons use `navigator.clipboard.writeText`, which is unaffected. This deters casual copying only; page source and developer tools remain accessible, as they always do on the web.
- **Downloads** go through the viewer's confirmation in the artifact, or a standard browser download elsewhere.

---

## 19. Troubleshooting

| Symptom | Cause and fix |
|---|---|
| Text menu shows "System heavy sans (raster)" | opentype.js did not load (offline or blocked CDN). Text falls back to raster tracing. Inline the library for offline use. |
| "WOFF2 is not supported" | opentype.js cannot read WOFF2. Use TTF, OTF or WOFF. |
| Photo does nothing or shows "Could not read that image" | Unsupported format (often HEIC outside Safari). Convert to JPG or PNG. |
| Upload button does nothing inside a mobile app | The host view may block file pickers. Open the page in a regular browser. |
| Save buttons say saving is not available | The viewer did not grant downloads. Use the Copy buttons, or open the file directly. |
| "Video recording is not supported" | The browser lacks MediaRecorder or WebM. Use Chrome, Edge or Firefox. |
| Very slow image tracing | Lower Detail resolution or Colours. |
| Too many tiny regions | Raise Smallest region kept, lower Colours, raise Speckle clean-up. |
| Equation shows "unknown name" | Only t, π, e and standard functions are allowed. Rename variables to t. |
| Equation looks wrong | Check the t range (untick Detect and set it). Desmos exports use 0 to 1. |
| Nothing visible after zooming very far | The view covers a tiny area; press Fit, or use Zoom into the last circle, which follows the pen. |
| 3D photo says "approximate depth" | The depth model could not download (blocked host, offline, or CSP). See §6. |
| No full-screen button | The browser or host does not allow full screen (iPhone Safari, some embedded viewers). |

---

## 20. Coding conventions

- One file, one IIFE, no globals except `EMBEDDED_FONTS`.
- Colours are CSS custom properties (light and dark sets); canvas colours are read through `readColors()` / `deriveInk()`.
- Typed arrays (`Float64Array`, `Int32Array`) for all numeric buffers.
- Interface copy uses plain, specific wording; errors say what went wrong and what to do.
- Every control has a stable `id`.

---

## 21. Licences

- Application code, sample images and sample text: original to this project.
- Embedded fonts: SIL Open Font License 1.1 (Archivo Black, Anton, Playfair Display, Rubik, Bricolage Grotesque).
- Interface fonts via Google Fonts: SIL Open Font License 1.1.
- opentype.js: MIT licence.
