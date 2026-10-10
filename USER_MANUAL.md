# Fourier Outline Tracer — User Manual

> Draw anything with spinning circles. Type a sentence, upload a photo, sketch with your finger or paste an equation, and the app traces it with Fourier epicycles: a chain of circles, each turning at a fixed speed, whose tip draws your picture.

This manual covers every button, slider, gesture and workflow in the app. If you want the maths and the project overview, read **README.md**. If you want to build, deploy or extend the code, read **INSTRUCTIONS.md**.

---

## Table of contents

1. [What the app does](#1-what-the-app-does)
2. [Getting started in 30 seconds](#2-getting-started-in-30-seconds)
3. [The screen at a glance](#3-the-screen-at-a-glance)
4. [The header bar](#4-the-header-bar)
5. [The canvas: gestures and keyboard](#5-the-canvas-gestures-and-keyboard)
6. [The playback bar (HUD)](#6-the-playback-bar-hud)
7. [Source tabs: Text, Image, Draw, Equation](#7-source-tabs)
8. [Quality](#8-quality)
9. [Explore: microscope, dive, follow, converge, sound, video, glow](#9-explore)
10. [Machine: circles vs X-Y, one machine vs per outline, 2D / 3D / 4D](#10-machine)
11. [Spectrum](#11-spectrum)
12. [Canvas settings: background, speed, circles, fill, outline, arrow, grid](#12-canvas-settings)
13. [Image tracing settings](#13-image-tracing-settings)
14. [Equation export](#14-equation-export)
15. [Export: SVG, PNG, coefficients](#15-export)
16. [How it works panel](#16-how-it-works-panel)
17. [Step-by-step workflows](#17-step-by-step-workflows)
18. [Tips for the best results](#18-tips-for-the-best-results)
19. [Phone and tablet use](#19-phone-and-tablet-use)
20. [Troubleshooting](#20-troubleshooting)
21. [FAQ](#21-faq)
22. [Glossary](#22-glossary)
23. [Copy protection](#23-copy-protection)
24. [Privacy](#24-privacy)

---

## 1. What the app does

Any closed outline can be written as a sum of circles:

```
z(t) = Σ cₖ · e^{i·k·t}
```

Each term is one circle. `|cₖ|` is its radius, `k` is how many turns it makes while the picture is drawn once, and `arg cₖ` is where it starts. Put the circles end to end, spin them, and the tip of the last one draws the shape.

The app:

- finds the outlines in your text, photo, sketch or equation;
- joins them into **one path**, with "pen-up" jumps between separate pieces;
- computes the circles with a Fast Fourier Transform (FFT);
- picks just enough circles to hit your chosen accuracy;
- animates the machine, fills the shapes in colour, and lets you export the result as a vector SVG, a 4K PNG, a video, coefficient data or a ready-to-paste equation.

The **main circle stays in the same place** for every input. Whether you draw one letter, a long sentence or a full photo, the machine sits at the same anchor, so every drawing feels like the same instrument.

---

## 2. Getting started in 30 seconds

1. Open the app. It starts on the **Text** tab with a sample sentence already drawing.
2. Type your own words in the text box. The drawing restarts automatically.
3. Press **Play / Pause** to stop and start.
4. Use **Speed** to slow down (down to 0.03×) or speed up (up to 16×).
5. Scroll on the canvas to zoom. Drag to move around.
6. When it looks right, open **Export** and press **Save SVG**.

For a photo, switch to the **Image** tab and press **Upload**. For your own sketch, use **Draw**. To recreate a curve from maths, use **Equation**.

---

## 3. The screen at a glance

| Area | Where | What it holds |
|---|---|---|
| Header | Top | Title, live formula, Compare (eye) button, zoom controls, full-screen button, and the ☰ menu button on phones |
| Canvas | Centre / top on phones | The drawing, the circles, the on-canvas tools |
| HUD (playback bar) | Under the canvas | Back, progress, forward, play, restart, finish, speed, counters (a compact one-row version on phones) |
| Side panel | Right on desktop; a slide-in ☰ menu on phones and small tablets | Source tabs and every settings section |

On wide screens the canvas and panel sit side by side. On phones and small tablets the canvas fills the screen and the panel becomes a menu that slides in from the right (see [Phone and tablet use](#19-phone-and-tablet-use)). Nothing overlaps or runs off the screen.

Panel sections can be opened and closed by tapping their titles.

---

## 4. The header bar

### Title and formula
The title is followed by a live formula line showing the series being drawn, for example `z(t) = Σ cₖ e^{ikt}` with the current number of circles.

### Compare with original (eye button)
Shown only for **uploaded images**. Press the eye to show the **original photo side by side** with the traced drawing. Press again to go back to the full-width drawing. Side-by-side is used on every screen size.

### Zoom controls
| Control | Action |
|---|---|
| **−** | Zoom out |
| **%** label | Current zoom. Shows small and huge values cleanly, e.g. `0.1%`, `500%`, `10,000,000%`, and beyond |
| **+** | Zoom in |
| **Fit** | Reset the view so the whole drawing fits the canvas |
| **⤢ Full screen** | Hides the browser bars (and turns a phone to landscape where allowed). Hidden where the browser doesn't support it |
| **☰ Menu** (phones) | Opens the settings menu |

Zoom ranges from extreme zoom-out (far below 1%) to an almost unlimited zoom-in (the view supports magnification up to 10¹⁴). Zoom changes **only the canvas**, never the page.

---

## 5. The canvas: gestures and keyboard

### 2D mode
| Input | Action |
|---|---|
| Mouse wheel / trackpad scroll | Zoom in or out around the pointer |
| Pinch (touch) | Zoom around your fingers |
| Drag (mouse or one finger) | Pan the view |
| Double-click / double-tap | Zoom in ×2 around that point |
| **+** / **=** | Zoom in |
| **−** | Zoom out |
| **0** | Fit (reset view) |

Keyboard keys work after you click the canvas once.

### Microscope on
| Input | Action |
|---|---|
| Wheel | Changes how deep the microscope looks into the chain |

### 3D and 4D modes
| Input | Action |
|---|---|
| Drag left / right | Turns the object the way you drag: drag right and it turns right, drag left and it turns left |
| Drag up / down | Tilts the object the way you drag: drag down and it tilts down (you see its top), drag up and it tilts up |
| Shift + drag, or right-drag | Pan |
| Two fingers (touch) | Drag to move it, pinch to zoom |
| Double-click / double-tap | Zoom in |
| Wheel / pinch | Zoom |

Dragging stops auto-orbit, so the solid stays where you put it.

### On-canvas drawing tools (Draw tab only)
When the **Draw** tab is active, a small tool strip appears on the canvas:

| Icon | Action |
|---|---|
| ✏️ Pen | Draw strokes |
| 🧽 Eraser | Erase parts of strokes by dragging over them |
| ↶ Undo | Remove the last stroke |
| 🗑 Clear | Remove everything you drew |

These icons are hidden on every other tab.

---

## 6. The playback bar (HUD)

| Control | What it does |
|---|---|
| **⏪ Back 5%** | Jumps the drawing back by 5% of the full run |
| **Progress slider** | Drag to scrub to any point. The drawn part, fills and dashed targets all update instantly |
| **⏩ Forward 5%** | Jumps forward by 5% |
| **Percentage** | How much of the drawing is done |
| **▶ / ⏸ / ↻** | Play, pause, or replay when finished |
| **Restart** | Clears the drawing and starts from 0% |
| **Finish** | Jumps to 100% so you see the completed picture at once |
| **Speed chip** | Opens the speed menu: **0.03×, 0.06×, 0.12×, 0.25×, 0.5×, 1×, 2×, 4×, 8×, 16×**. The chip is coloured so it is easy to spot |
| **Outline counter** | Which outline is being drawn, out of how many (e.g. `12 / 34`) |
| **t** | Current time in radians (0 → 2π for one full drawing) |

**On phones** the bar is one row: **▶ / ⏸** icon (left of the progress bar), the **progress bar**, the **↻ restart** icon (right of it), the percentage, and a **speed button** showing the current speed (e.g. `1×`). Tapping the speed button opens a small pop-up with all speeds; tap one to choose it, or tap outside to close. Back/forward 5%, Finish and the counters are left out on phones to keep it on one line.

In the **Draw** tab, while you are still drawing, play (or restart) turns your strokes into circles and starts the animation.

---

## 7. Source tabs

There are four ways to feed the machine. Switching tabs loads that source and restarts the animation.

### 7.1 Text
| Control | Description |
|---|---|
| **Text box** | Type any words or sentences. Line breaks are kept. Long sentences wrap |
| **Font** | Five built-in bold display fonts: Archivo Black, Anton, Playfair Display Black, Rubik Black, Bricolage Grotesque ExtraBold |
| **Your font…** | Load your own `.ttf`, `.otf` or `.woff` file. Its exact glyph curves are used |
| **Colour per letter** | Each letter gets its own colour from the palette |
| **Mono colour** | One colour for all letters (pick it with the swatch) |
| **Wrap** | Slider for line width: how many characters before text wraps to the next line |

Text is read from the font's real Bézier curves, not from pixels, so letters stay razor-sharp at any zoom.

### 7.2 Image
| Control | Description |
|---|---|
| **Upload** | Choose a photo (JPG, PNG, WebP, GIF, BMP…). You can also drag and drop a file onto the canvas |
| **Compare with original** | Same as the eye button: original and drawing side by side |

The app groups similar colours, finds the borders between them with sub-pixel precision, and traces every region, including small details and holes. Tune this under [Image tracing settings](#13-image-tracing-settings).

### 7.3 Draw
| Control | Description |
|---|---|
| **Canvas** | Draw directly with mouse, pen or finger |
| **Play (playback bar)** | Turns your strokes into circles and starts drawing. Restart and the progress bar work the same as for text |
| **Ready-made drawings** | Horse, Garden, Sunset, Flower, Butterfly, Sailboat: loaded as editable strokes. Change them with the pen and eraser, then press play |
| **Shapes drawn** | How many separate strokes (shapes) are on the canvas, e.g. "12 shapes drawn". Hidden while the canvas is empty |
| **Edit drawing** | Go back from the animation to your strokes to change them, then animate again |

Pen, eraser, undo and clear are on the canvas itself while this tab is open.

### 7.4 Equation
| Control | Description |
|---|---|
| **Equation box** | Paste or type an equation; it draws by itself about 0.7 s after you stop typing. The app accepts its own format and Desmos, GeoGebra, Mathematica and Python equations |
| **t from / t to** | The parameter range. Leave **detect** ticked to take it from the equation; typing a value unticks it |
| **Detect** | Reads the t range from the pasted equation. The status line shows which format was recognised |
| **Examples** | 9 ready equations: Rose (a real rose, 7 curves), Heart, Butterfly, Spirograph, Starfish, Trefoil knot, Cardioid, Nephroid, Lissajous |

You can copy an equation from **Equation export**, paste it here, and the same drawing comes back by itself. Several curves in one paste (one per outline) are rebuilt as separate shapes.

Only maths is accepted: numbers, `t`, `+ − × ÷ ^`, brackets, `sin cos tan exp log sqrt abs`, `pi`, `e` and `i`. Anything else is rejected, so pasted text cannot run code.

---

## 8. Quality

| Preset | Accuracy target | Use it for |
|---|---|---|
| **Draft** | 0.5% of the drawing height | Quick previews, slow devices |
| **HD** (default) | 0.1% | Everyday use |
| **Ultra HD** | 0.025% | Long sentences, detailed photos, posters |
| **Manual** | You choose | Exact control with the **N** slider |

- **N slider** (Manual): the number of circles. Fewer circles give a softer, "wobbly" look, which is great for showing how the approximation improves.
- **Lanczos smoothing**: softens the ripples ("ringing") that appear at sharp corners when you use few circles.
- **Metrics**: shows the number of circles used, the guaranteed maximum error (no point on the drawing is further from the true outline than this), the FFT size, and the build time.

The error figure is a real upper bound: it is the sum of the radii of every circle left out, so the true error can only be smaller.

---

## 9. Explore

| Feature | What it does |
|---|---|
| **Microscope** | A round inset that magnifies the innermost circles at the tip of the chain, so you can watch the tiny ones spin. Use the wheel over the canvas to change depth |
| **Dive** | Smooth automatic zoom dive into the pen tip |
| **Zoom into the last circle** | The camera flies in and follows the pen, zooming until the very last, smallest circle fills the view. That circle is highlighted. The animation slows down so it stays watchable at that scale |
| **Stop following** | Same button while following. Returns control to you |
| **Last circle: X machine / Y machine** | Shown in X-Y mode. Chooses which machine's last circle the camera follows. Switch while following and the camera flies across |
| **Depth** | How far in "Zoom into the last circle" goes |
| **Watch it converge** | Replays the drawing with 1, 2, 3 … circles, so you can see the picture appear from a blob into the exact outline |
| **Hear the shape** | Plays the path as stereo sound: x on the left channel, y on the right. Feed it to an oscilloscope in X-Y mode and it draws your picture |
| **Pitch** | Changes the tone of Hear the shape |
| **Record video** | Records the canvas as a WebM video. Press again to stop and save |
| **Neon glow** | Adds a glowing light trail to the pen |

### Seeing the very last circle
The circles are calculated in full double precision. Even at enormous zoom levels (10,000,000% and far beyond) the app redraws everything exactly around the view, so the smallest circle is a crisp ring, not a blur. Press **Zoom into the last circle** to get there automatically.

---

## 10. Machine

### Circle style
| Option | Description |
|---|---|
| **Circles** | Classic epicycles: one chain of complex circles drawing x and y together |
| **X-Y** | Two machines: one at the top draws x, one on the side draws y. Their lines cross at the pen. This is the style from the "knight" reels |

### Machine scope
| Option | Description |
|---|---|
| **One fixed machine** | The whole picture is one path drawn by one chain. The main circle never moves between inputs |
| **Per outline** | Each outline gets its own small set of circles. Builds fast and keeps detail even with very many shapes |

### Dimension
| Option | Description |
|---|---|
| **2D** | Normal (x, y) drawing |
| **3D** | Real (x, y, z). The outline is turned into a closed space curve: front face, edge, back face. Each axis gets its own Fourier series, so every harmonic is an ellipse in 3D space. The result is shaded and drawn in perspective |
| **4D** | Real (x, y, z, time). A second time variable τ slowly morphs the 3D curve into a target and back: r(t, τ) = Σ Σ c(k, n) e^{i(kt + nτ)} |

3D / 4D controls:

| Control | Description |
|---|---|
| **Depth of solid** | How thick the 3D extrusion is (for photos with real depth: how tall the relief is) |
| **Edge: Flat / Bevel / Rounded** (3D) | The cross-section of the solid. Flat is a plain prism. **Bevel** draws every outline four times (two laps stepped in on each face) so the chamfer is part of the same Fourier machine; **Rounded** uses six laps for a curved edge. More laps need more circles for the same accuracy. The bevel is automatically limited to a fraction of each letter's stroke width, so thin parts never fold over |
| **Edge size** (3D) | How wide the bevel or rounding is, as a share of half the thickness |
| **Real depth (AI)** (photos) | An AI model (Depth Anything V2 Small) estimates how far away every pixel is, on your device. The depth map becomes a 2D Fourier surface z(u,v) = Σ Σ d(m,n) e^{i(mu+nv)}. Every outline is lifted onto it, and one x(t), y(t), z(t) machine draws the result. The first use downloads the model (~27 MB, then cached). If it can't load, a clearly labelled approximate depth is used |
| **Extruded solid** (photos) | The older look: every outline pushed straight out into a slab |
| **Surface harmonics** | How many 2D Fourier terms describe the depth surface ((2K+1)² terms). Low = smooth rolling hills, high = fine detail |
| **Depth contour lines** | Iso-depth lines (like a contour map) added to the drawing and traced by the same machine, coloured from far to near |
| **Show the Fourier depth surface** | Draws the surface as a light wire mesh. A preview of the depth map sits under it (bright = near) |
| **Paint the photo on the depth surface** | Draws the depth surface as a lit 3D mesh with your photo as its texture. It fades in as the outlines are traced. Turn it off for the older outlines-only look |
| **Rise from the flat photo** (4D, photos) | Time τ lifts the flat photo into its real depth and back: r(t, τ) = flat + s(τ)·depth |
| **Auto-orbit** | Slowly turns the camera around the object |
| **Morph target** (4D) | **Ring** (default), **Text**, or **Sequence** |
| **Morph text** (4D) | The text to morph into. In **Sequence** mode, separate shapes with a vertical bar, for example `star \| HELLO \| heart \| ring`. Built-in shapes: ring, square, triangle, star, heart. Anything else is drawn as text. The cycle ends back at your own drawing |
| **Pause on each shape** (4D) | How long the morph rests on every shape before moving on |
| **Travelling wave height / Waves along the object** (4D) | Adds a wave that runs through the object, z += h·sin(2π·n·x/W − τ). Its spectrum is added to the circle chain, so the circles still end exactly at the pen |
| **Speed of time** (4D) | How fast τ moves |
| **Time τ** (4D) | Scrub τ by hand |
| **Time trail** (4D) | Shows faded copies of earlier moments, so you see the shape's path through time |

---

## 11. Spectrum

A bar chart of circle sizes against frequency. Big bars on the left are the slow, large circles that set the overall shape. The long tail of small bars on the right adds the fine detail. Use it to see how much each frequency contributes.

---

## 12. Canvas settings

| Control | Description |
|---|---|
| **Background** | Colour swatches plus a custom colour picker. Lines and circles adjust so they stay visible |
| **Drawing speed** | The same as the Speed chip in the HUD |
| **Circles drawn** | How many circles are shown on screen (the maths always uses all of them). Lower it to clean up the view |
| **Fill style: Solid** | Shapes fill with flat colour once their outline is done |
| **Fill style: Drawn lines** | Shapes fill with hatch lines that are *drawn* inside the outline, like a pen shading. No automatic colour flood |
| ‣ **Line count** | Number of hatch lines (default 60) |
| ‣ **Thickness** | Hatch line width (default 1.2) |
| ‣ **Angle** | Hatch line angle (default 45°) |
| **Fill style: Outline only** | No fill, just the traced lines |
| **Show target outline** | Dashed outlines of shapes still to draw. Each one stays visible **until its whole shape is finished**, even when the pen moves away to draw something underneath |
| **Direction arrow** | A small arrow just ahead of the pen showing where it will move next |
| **Show grid** | Turns the background grid on or off |

Fills wait until every part of a shape (its outer edge and all its holes) is complete, so you never see half-filled letters such as an "O" with its hole flooded.

---

## 13. Image tracing settings

These apply to uploaded images. Changes re-trace the image.

| Control | Description |
|---|---|
| **Colours** | How many colour groups the photo is split into. More colours give more regions and detail |
| **Detail resolution** | 200–2400 px. The size the image is analysed at. Higher catches finer detail but takes longer |
| **Smallest region** | Regions smaller than this are ignored. Raise it to drop noise |
| **Edge smoothing** | Smooths the traced borders. Lower keeps sharp corners |
| **Speckle clean-up** | Removes isolated stray pixels before tracing |
| **Leave out background** | Skips the largest edge-touching colour, usually the background |

---

## 14. Equation export

| Control | Description |
|---|---|
| **Format** | **App**, **Desmos**, **GeoGebra**, **Mathematica**, **Python** |
| **Scope** | **Per outline** (one equation per shape) or **Whole** (one equation for the full path) |
| **Circles per equation** | How many terms to include. More terms = more exact, longer equation |
| **Output box** | The finished equation(s) |
| **Copy** | Copies the full equation to your clipboard |
| **Links** | Opens the matching simulator website (e.g. Desmos, GeoGebra) so you can paste it there |

How to use each format:

- **Desmos**: paste into desmos.com/calculator. Uses t from 0 to 1.
- **GeoGebra**: paste into the GeoGebra input bar.
- **Mathematica**: paste into a notebook and evaluate; it uses `ParametricPlot`.
- **Python**: paste into a `.py` file or notebook; it uses NumPy and Matplotlib.
- **App**: paste back into the **Equation** tab; it draws by itself.

**In 3D (and 4D)** every curve also gets a third equation, **z(t)**, pointing towards you:

- **Extruded solid** (text, drawings, equations): z(t) is the front face, and the header line `# 3D · extruded solid, depth 40%` records the depth. Pasting it back into the Equation tab rebuilds the same solid in 3D.
- **Real depth relief** (photos): z(t) is a Fourier series of the depth along each outline. Pasting it back rebuilds the lifted outlines in 3D (the depth contour lines and surface mesh are not included).
- Desmos: use the **Desmos 3D** calculator. GeoGebra: the **3D** calculator (`Curve(x, y, z, t, 0, 2π)`). Mathematica: `ParametricPlot3D`. Python: a Matplotlib 3D plot.
- In 4D the equation lists every shape of the cycle as x(t), y(t), z(t) plus the blend `r = r₁ + (r₂ − r₁)·s(τ)`, `s = (1 − cos τ)/2`, with a τ slider (Desmos 3D, GeoGebra, Mathematica Manipulate) or an animated Matplotlib loop (Python).

Long equations are cut in the preview box, but **Copy** always copies all of it. When pasting a coloured equation back, filling comes back on automatically.

---

## 15. Export

| Control | Description |
|---|---|
| **Include background** | Adds your background colour to the export. Off = transparent |
| **Save SVG** | Coloured vector file with smooth Bézier curves. Scales to any size without blur |
| **PNG 4K** | High-resolution image |
| **Copy SVG** | Copies the SVG code to the clipboard |
| **Copy coefficients** | Copies the circle data (frequency, radius, phase) as JSON |

---

## 16. How it works panel

A short in-app explainer of the method: how outlines are found, how the circles are calculated, how many are kept, and what the error figure means.

---

## 17. Step-by-step workflows

### A. Draw a sentence and save it as a vector
1. Text tab → type your sentence.
2. Pick a font. Choose colour per letter or mono.
3. Quality → **Ultra HD** for long sentences.
4. Press **Finish** (or let it play).
5. Export → **Save SVG**.

### B. Trace a photo
1. Image tab → **Upload**.
2. Press the **eye** to compare with the original.
3. Too much noise? Raise **Smallest region** or turn on **Speckle clean-up**.
4. Missing detail? Raise **Colours** and **Detail resolution**.
5. Background drawn when you don't want it? Turn on **Leave out background**.
6. Export as SVG or PNG 4K.

### C. Animate your own sketch
1. Draw tab → sketch with the pen. Use the eraser or undo to fix mistakes.
2. Press **Play** on the playback bar.
3. Want changes? **Edit drawing**, adjust, animate again.

### D. Move a drawing into Desmos
1. Equation export → Format **Desmos**, Scope **Per outline**.
2. **Copy** → open Desmos with the link → paste.

### E. Recreate a drawing from an equation
1. Copy an equation (from this app or from Desmos, GeoGebra, Mathematica, Python).
2. Equation tab → paste. It draws by itself a moment later.

### F. Hand-shaded look
1. Canvas → Fill style **Drawn lines**.
2. Tune **Line count**, **Thickness** and **Angle**.

### G. See the smallest circle
1. Press **Zoom into the last circle**.
2. Lower the speed to 0.03× to watch it closely.
3. Press **Stop following** to return.

### H. Explore 3D and 4D
1. Machine → **3D**. Drag to turn it (it follows your hand); set **Depth of solid**. For a photo, **Real depth (AI)** lifts it onto its estimated depth.
2. Machine → **4D**. Watch it morph through time; switch **Morph target** to **Text** and type a word, or to **Sequence** and type `star | heart | ring`; turn on **Time trail**; raise **Travelling wave height**.
3. Export: in 3D, **Save 3D model (GLB)** (colours; opens in Blender, 3D viewers, AR) or **Save STL (3D printing)**. In 4D, **Save looping GIF** renders one full period of time with one full turn of the camera, so it loops seamlessly.

### I. Record a video
1. Explore → **Record video**.
2. Let it draw.
3. Press again to stop and save.

---

## 18. Tips for the best results

- Use **Ultra HD** for long text or detailed images; **Draft** while experimenting.
- High-contrast photos with clear subjects trace best.
- For logos, 2–6 **Colours** is usually ideal; for photos, 8–16.
- Lower **Circles drawn** if the screen gets crowded; the drawing stays exact.
- Slow speeds (0.12×, 0.06×, 0.03×) show the motion of each circle beautifully.
- Turn on the **Direction arrow** to follow the pen easily.
- Dark backgrounds with **Neon glow** make the best videos.

---

## 19. Phone and tablet use

- The canvas fills the screen. All settings are in the **☰ menu** (top right), which slides in from the side. Close it with **✕**, by tapping outside it, or by swiping it to the right. It closes by itself when you pick Draw, a sample, Microscope, Dive or Zoom into the last circle, so you can see the result.
- **Full screen** (the ⤢ button) hides the browser bars and turns the screen to landscape where the phone allows it. On first opening in portrait, a prompt offers this once. Browsers only allow full screen after a tap, and iPhones don't support it, so there the button is hidden.
- The playback bar is one compact row: play/pause icon, progress bar, restart icon, percentage and a speed button with a small pop-up (see [The playback bar](#6-the-playback-bar-hud)).
- Pinch to zoom, one finger to pan, double-tap to zoom in. In 3D/4D, one finger turns the object the way you drag and two fingers move it.
- Buttons are larger on touch screens, and photos are traced at 800 px detail by default so they build faster.
- In Draw mode, your finger is the pen. The on-canvas tool strip gives pen, eraser, undo and clear.
- Very large images and Ultra HD may build slower on phones. Use HD or Draft if needed.

---

## 20. Troubleshooting

| Problem | Fix |
|---|---|
| Image won't load | Use a common format (JPG, PNG, WebP). Try a smaller file |
| Drawing looks blobby | Raise quality, or turn off Manual mode with a low N |
| Too many tiny shapes | Raise **Smallest region**, turn on **Speckle clean-up**, lower **Colours** |
| Background traced | Turn on **Leave out background** |
| Ripples at corners | Turn on **Lanczos smoothing** or raise quality |
| Slow on my device | Use **Draft**, lower **Detail resolution**, use **Per outline** |
| 3D photo says "approximate depth" | The AI depth model couldn't download (offline, or blocked by the page host such as the claude.ai preview). Open the app on its own hosting, or use **Extruded solid** |
| No full-screen button | Your browser doesn't allow pages to go full screen (iPhone Safari, some app views) |
| Lost on the canvas | Press **Fit** or the **0** key |
| Pasted equation fails | Check the format with **Detect**. Remove non-maths text. Set **t from / t to** if not auto |
| Recording doesn't start | Your browser may not support WebM recording (some Safari versions). Try Chrome, Edge or Firefox |
| No sound | Click on the page first; browsers block sound until you interact |
| Download doesn't start | Allow downloads for the site in your browser |

---

## 21. FAQ

**Is the drawing really vector?**
Yes. The curve is a sum of circles, which is an exact mathematical formula. The canvas redraws it at whatever zoom you choose, and SVG export uses smooth Bézier curves.

**How many circles does it use?**
As many as needed for the chosen accuracy. It can be a few dozen for a simple letter up to many thousands for a detailed photo. The Quality metrics show the exact count.

**Why does the main circle never move?**
The drawing is always scaled and centred into the same frame, so the first circle sits at the same anchor for every input.

**How does it know where the borders are in a photo?**
It groups pixels by perceived colour (in CIELAB space), then finds the lines where one group meets another, with sub-pixel accuracy.

**How does it know which part is drawn on top?**
Shapes are grouped by containment (outer shape with its holes). Each group fills only after all its parts are drawn, and the dashed target stays until the group is complete.

**What are 3D and 4D here?**
3D means three coordinates (x, y, z) each with its own Fourier series. 4D adds a real time coordinate τ, so the 3D curve changes over time.

**Can I use my own font?**
Yes: Text tab → **Your font…**.

**Does anything get uploaded?**
No. Everything runs inside your browser.

---

## 22. Glossary

| Term | Meaning |
|---|---|
| **Epicycle** | A circle riding on another circle |
| **Fourier series** | Writing a shape as a sum of circles (sines and cosines) |
| **Coefficient cₖ** | One circle's size and starting angle |
| **Frequency k** | How many turns a circle makes per drawing |
| **FFT** | Fast Fourier Transform, the algorithm that computes the circles |
| **N** | Number of circles used |
| **Outline** | One closed border of a shape |
| **Pen-up** | A jump between separate outlines, not drawn |
| **Lanczos σ** | Smoothing that reduces ripples at corners |
| **CIELAB** | A colour space that matches human colour perception |
| **Marching squares** | The method that finds borders between colour regions |
| **Hatch fill** | Filling a shape with drawn parallel lines |
| **τ (tau)** | The time coordinate in 4D |

---

## 23. Copy protection

Interface text cannot be selected or copied: headings, titles, button labels, placeholders, status messages and panel text. Right-click menus and dragging are disabled outside the drawing area.

Text boxes still work normally. You can type, select, copy and paste inside the text, equation and morph-text fields, and the Copy buttons still copy equations, SVG and coefficients.

This is a deterrent, not absolute protection: anyone can still view the page source in their browser's developer tools.

---

## 24. Privacy

All processing happens locally in your browser. Images, text, drawings and equations are never sent to a server. Downloads are saved directly to your device.
