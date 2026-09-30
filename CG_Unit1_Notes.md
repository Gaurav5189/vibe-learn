# Computer Graphics — Unit I Notes
**MCA Sem 3 | Elective A07 (MCC-2.3.4) | Ravenshaw University**
**Textbook:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed., Pearson

---

## 1. A Survey of Computer Graphics (Applications)

**Computer Graphics (CG):** creation, storage and manipulation of images/pictures using a computer.

| Application | Brief Explanation | Examples |
|---|---|---|
| **Computer-Aided Design (CAD)** | Design of buildings, cars, circuits, aircraft using interactive graphics; objects shown as wire-frame or shaded models, and can be tested/simulated | Architectural plans, VLSI layout, AutoCAD |
| **Presentation Graphics** | Converts numeric data into charts/graphs for reports and slides | Bar chart, pie chart, line graph, PowerPoint |
| **Computer Art** | Fine art and commercial art made with paint programs and graphics packages | Digital painting, logos, advertisements |
| **Entertainment** | Movies, TV, music videos and games | Animation, special effects, video games |
| **Education & Training** | Simulators and visual models for learning | Flight simulator, driving simulator, medical training |
| **Visualization** | Graphical display of large/complex data to find patterns. *Scientific visualization* = physical data; *Business visualization* = commercial data | Weather maps, MRI, stock trends |
| **Image Processing** | Modifies/analyzes existing images (input is a picture, unlike CG where input is a description). Used for enhancement, recognition | Medical imaging, satellite images, face recognition |
| **Graphical User Interface (GUI)** | Interface using windows, icons, menus and pointer (WIMP) | Windows, Android, macOS |

> **Note:** CG = picture *synthesis* from data; Image Processing = picture *analysis/modification* of an existing picture.

---

## 2. Overview of Graphics Systems

### 2.1 Video Display Devices
Most common output device = **CRT (Cathode Ray Tube)** monitor.

**Refresh CRT – Components**
1. **Electron gun** – cathode (heated filament) emits electrons.
2. **Control grid** – controls number of electrons → controls **intensity (brightness)**.
3. **Focusing system** – makes electrons converge to a small spot (electric/magnetic).
4. **Deflection plates/coils** – horizontal and vertical; steer beam to a screen position.
5. **Phosphor-coated screen** – glows when hit by electrons.

**Working:** Electrons strike the phosphor → light emitted for a short time (**persistence**). Since glow fades quickly, the picture must be redrawn repeatedly = **refreshing** (usually **60 frames/sec or more**).

**Key terms**
- **Persistence:** time a phosphor continues to emit light after the beam is removed (time for brightness to fall to 1/10 of original).
- **Refresh rate:** number of times per second the picture is redrawn.
- **Resolution:** max number of points (pixels) displayable without overlap.
- **Aspect ratio:** ratio of horizontal to vertical points (or length) of the display. E.g. 4:3.
- **Pixel (picture element):** smallest addressable screen point.

**Color CRT:** uses three phosphors (Red, Green, Blue). Two methods:
- **Beam-penetration:** two phosphor layers (red, green); beam speed decides color. Limited colors.
- **Shadow-mask:** three electron guns (R, G, B) + a shadow mask with holes so each beam hits only its own dot. Gives wide range of colors (used in raster systems).

**Other display devices (brief):** Direct-view storage tube (DVST), Flat-panel displays (LCD, LED, plasma – thinner, less power), 3D viewing devices.

---

### 2.2 Raster Scan Systems (and Displays)

**Raster-scan display:** beam sweeps the screen **left to right, top to bottom**, row by row (each row = **scan line**). Intensity is turned on/off to make pixel pattern.

- Picture is stored in **Frame Buffer (Refresh Buffer)** – memory holding intensity values for all pixels.
- **Retrace:** *Horizontal retrace* = beam returns to start of next line; *Vertical retrace* = beam returns from bottom-right to top-left.
- **Interlacing:** odd lines displayed first, then even lines – reduces flicker at lower refresh rates.
- **Bit map** = frame buffer with 1 bit/pixel; **Pixmap** = more than 1 bit/pixel.
- Frame-buffer size = *resolution × bits per pixel*. E.g., 1024×768 at 24 bpp = 1024×768×3 bytes ≈ 2.25 MB.

**Raster system architecture**
- **Video controller:** reads frame buffer and generates beam control signals; refreshes screen.
- **Display processor (graphics controller):** takes load off the CPU – does scan conversion of primitives, color, etc.

| Advantages | Disadvantages |
|---|---|
| Realistic images, shading, many colors | Stair-step (jagged) lines → **aliasing** |
| Low cost, TV-like technology | Needs large memory for frame buffer |
| Can fill areas with patterns/colors | Scan conversion required |

---

### 2.3 Random Scan Systems (Vector / Stroke / Calligraphic Displays)

- Beam is directed **only to the parts of the screen where the picture is to be drawn**, line by line, not whole screen.
- Picture is stored as a set of **line-drawing commands** in a **display file (display list / refresh display file)** – not as pixels.
- System cycles through the display list to refresh each line.
- Refresh rate depends on number of lines (30–60 times/sec).

| Advantages | Disadvantages |
|---|---|
| High resolution, smooth lines | Poor for realistic shaded scenes |
| Less memory | Cannot fill areas with patterns easily |
| Easy to edit/modify lines | Flicker if many lines |

**Raster vs Random Scan**

| Basis | Raster Scan | Random Scan |
|---|---|---|
| Beam movement | Whole screen, row by row | Only along picture lines |
| Storage | Frame buffer (pixels) | Display file (commands) |
| Resolution | Lower (pixel-based) | Higher |
| Line quality | Jagged (aliasing) | Smooth |
| Realism/shading | Good | Poor |
| Memory | More | Less |
| Example | TV, monitors | Pen plotter, vector display |

---

### 2.4 Input Devices
Devices that feed data/commands to graphics system.

| Device | Function |
|---|---|
| **Keyboard** | Text, commands, numeric data; cursor keys |
| **Mouse** | Moves cursor; buttons to select/pick |
| **Trackball / Spaceball** | Rolling ball moves cursor; spaceball gives 3D movement (push/pull/twist) |
| **Joystick** | Stick controls cursor direction |
| **Data glove** | Tracks hand/finger movement (VR) |
| **Digitizer / Graphics tablet** | Stylus draws on a flat surface → gives coordinates |
| **Touch panel** | Touch screen position detected (optical, electrical, acoustic) |
| **Light pen** | Pen-shaped device detects light from CRT spot to pick a screen position |
| **Scanner** | Converts drawings/photos to digital form |
| **Voice system** | Speech commands |

---

### 2.5 Hard-copy Devices
Produce permanent (paper/film) output.

- **Printers:**
  - *Impact* (dot-matrix) – pins strike ribbon.
  - *Non-impact* – **laser** (laser + toner), **inkjet** (sprays ink droplets), **thermal**.
- **Plotters:** draw vector graphics using pens – **drum** plotter, **flatbed** plotter, electrostatic/inkjet plotters. Used for engineering drawings (CAD).
- Also: film recorders, photo printers.

---

### 2.6 Graphics Software
Two categories:
1. **General Programming Packages** – library of graphics functions used inside a programming language (C, C++, Java). Functions for output primitives, attributes, transformations, viewing. Examples: **OpenGL, GKS, PHIGS, PHIGS+**.
2. **Special-purpose Application Packages** – for users who don't program. Examples: paint programs, CAD packages, CorelDRAW, Photoshop, AutoCAD.

**Coordinate representations:** objects are defined in **world coordinates** → mapped to **device (screen) coordinates**.
**Software standards:** **GKS** (Graphical Kernel System), **PHIGS** – ISO/ANSI standards for portable graphics.

---

## 3. Output Primitives
**Output primitives** = basic geometric structures used to describe a picture: *points, lines, circles, ellipses, polygons (fill areas), curves, text.*

### 3.1 Points and Lines
- **Point plotting:** converts a coordinate position from the application into device operations (set pixel intensity in the frame buffer).
- **Line drawing:** calculate intermediate points between two end points, then turn on the nearest pixels.
- **Scan conversion (rasterization):** process of converting a geometric primitive into pixel positions.
- Line equation: **y = m·x + b**, slope **m = Δy / Δx**, intercept **b = y₁ − m·x₁**.
- Problem: pixels are at integer positions → line gets jagged (**aliasing**).

---

### 3.2 DDA (Digital Differential Analyzer) Algorithm
Incremental method using the slope; samples the line at unit intervals in one coordinate.

**Steps** (endpoints (x₁,y₁), (x₂,y₂)):
1. dx = x₂ − x₁, dy = y₂ − y₁
2. steps = max(|dx|, |dy|)
3. Xinc = dx / steps, Yinc = dy / steps
4. Set (x, y) = (x₁, y₁); plot it.
5. Repeat `steps` times: x = x + Xinc; y = y + Yinc; plot (round(x), round(y)).

**Example:** (2,2) to (6,5): dx=4, dy=3, steps=4, Xinc=1, Yinc=0.75
Points: (2,2), (3,2.75→3), (4,3.5→4), (5,4.25→4), (6,5)

| Pros | Cons |
|---|---|
| Simple, faster than using y=mx+b directly | Floating-point arithmetic and rounding |
| No multiplication per step | Round-off error accumulates → line drifts on long lines |

---

### 3.3 Bresenham's Line Drawing Algorithm
Uses **only integer arithmetic**; a **decision parameter** picks which of two candidate pixels is closer to the true line.

**For slope 0 < m < 1** (left endpoint = (x₀,y₀)):
1. Δx = x₂−x₁, Δy = y₂−y₁ (use magnitudes).
2. Plot first point (x₀, y₀).
3. Initial decision parameter: **p₀ = 2Δy − Δx**
4. For each xₖ along the line:
   - if **pₖ < 0**: next pixel = **(xₖ+1, yₖ)**, **pₖ₊₁ = pₖ + 2Δy**
   - else: next pixel = **(xₖ+1, yₖ+1)**, **pₖ₊₁ = pₖ + 2Δy − 2Δx**
5. Repeat step 4 for Δx times.

For **m > 1**: swap roles of x and y. For negative slopes: use magnitudes and decrement y instead of increment.

**Example:** (20,10) to (25,13): Δx=5, Δy=3, p₀=1, 2Δy=6, 2Δy−2Δx=−4

| k | pₖ | Next pixel |
|---|---|---|
| 0 | 1 | (21,11) |
| 1 | −3 | (22,11) |
| 2 | 3 | (23,12) |
| 3 | −1 | (24,12) |
| 4 | 5 | (25,13) |

**DDA vs Bresenham**

| DDA | Bresenham |
|---|---|
| Floating point, rounding | Integer only |
| Slower | Faster |
| Less accurate (drift) | More accurate |
| Uses multiplication/division | Uses add/subtract only |

---

### 3.4 Midpoint Circle Algorithm
Circle: **x² + y² = r²** (center at origin; shift by (x꜀, y꜀) afterward).

**Key idea:** circle has **8-way symmetry** → compute one octant (from (0,r) to x = y, i.e. 90° → 45°), plot other 7 by symmetry: (±x,±y), (±y,±x).

**Circle function:** f(x,y) = x² + y² − r²
- f < 0 → point **inside** circle
- f = 0 → point **on** circle
- f > 0 → point **outside** circle

At each step, test the **midpoint** between two candidate pixels (xₖ+1, yₖ) and (xₖ+1, yₖ−1); its sign decides the pixel.

**Steps:**
1. Start (x₀, y₀) = (0, r).
2. Initial parameter: **p₀ = 5/4 − r** (≈ **1 − r** for integer r).
3. At each xₖ:
   - if **pₖ < 0**: next = (xₖ+1, yₖ), **pₖ₊₁ = pₖ + 2xₖ₊₁ + 1**
   - else: next = (xₖ+1, yₖ−1), **pₖ₊₁ = pₖ + 2xₖ₊₁ + 1 − 2yₖ₊₁**
4. Plot 7 symmetric points; move each to center: x = x + x꜀, y = y + y꜀.
5. Repeat until **x ≥ y**.

**Example:** r = 10, center (0,0): p₀ = 1 − 10 = −9

| k | pₖ | (xₖ₊₁, yₖ₊₁) |
|---|---|---|
| 0 | −9 | (1,10) |
| 1 | −6 | (2,10) |
| 2 | −1 | (3,10) |
| 3 | 6 | (4,9) |
| 4 | −3 | (5,9) |
| 5 | 8 | (6,8) |
| 6 | 5 | (7,7) → stop |

---

### 3.5 Filled Area Primitives
A **fill area** is usually a **polygon**; fill with solid color or pattern.

**(a) Inside–Outside Tests** (to decide if a point is inside a polygon)
- **Odd-even rule:** draw a line from the point to far outside; if it crosses an **odd** number of edges → inside.
- **Non-zero winding number rule:** count how many times the polygon winds around the point; **non-zero** → inside.

**(b) Scan-line Polygon Fill**
For each scan line: find intersections with polygon edges → sort by x → fill pixels between pairs of intersections (1st–2nd, 3rd–4th …). Special handling of vertices (count once/twice depending on whether edges are on opposite/same sides) and horizontal edges. Uses **edge table / active edge list** and coherence (Δx = 1/m per scan line).

**(c) Boundary-Fill Algorithm**
Start at an **interior point**; paint neighbors until a **boundary color** is reached. Works for regions with a **single-color boundary**.
- **4-connected:** neighbors = left, right, up, down.
- **8-connected:** also 4 diagonals (fills complex shapes correctly).
```
boundaryFill(x, y, fill, boundary):
    if getPixel(x,y) != boundary and getPixel(x,y) != fill:
        setPixel(x,y,fill)
        boundaryFill(x+1,y), (x-1,y), (x,y+1), (x,y-1)
```

**(d) Flood-Fill Algorithm**
Replaces a **specified interior color** (region not bounded by one color) with a new color, spreading to neighbors (4- or 8-connected) having the old color.

| Boundary-fill | Flood-fill |
|---|---|
| Stops at boundary color | Replaces old interior color |
| Boundary must be single color | Region can have multiple boundary colors |

> Recursive versions can overflow stack → use stack-based / scan-line seed fill.

---

## 4. Attributes of Output Primitives
**Attribute:** a parameter that controls **how** a primitive is displayed (color, size, style, etc.).
Two ways to handle: (i) pass attributes in the function parameter list, (ii) keep a **system list of current attribute values** and use separate functions to set them (used in GKS/PHIGS/OpenGL).

### 4.1 Line Attributes
- **Line type:** solid, dashed, dotted, dash-dotted (set with pixel mask / dash pattern).
- **Line width:** 1, 2, 3… pixels. For |m|<1 replicate pixels **vertically**; for |m|>1 replicate **horizontally**.
- **Line color:** intensity/color of pixels.
- **Pen and brush options:** shape and pattern of the drawing tool.
- **Line caps:** end shape – **butt** (flat, perpendicular), **round**, **projecting square**.
- **Line joins:** how two lines meet – **miter** (sharp), **round**, **bevel** (cut).

### 4.2 Curve Attributes
Same as lines: **color, width, dash/dotted style, pen/brush**. Width is obtained by plotting **additional pixels** (horizontal spans where |slope|<1, vertical where |slope|>1). Also **parallel curves** can be used for thick curves.

### 4.3 Color and Grayscale Levels
- **Color table (look-up table, LUT):** frame buffer stores **index** into the table; table holds actual RGB values. Saves memory (e.g., 8 bits → 256 colors from a palette of 16 million).
- **Direct (RGB) storage:** each pixel stores R, G, B values (e.g., 24-bit true color).
- **Grayscale:** for monochrome output, colors converted to **gray levels**. Gray level is computed as a weighted sum of R, G, B (commonly Y = 0.299R + 0.587G + 0.114B).
- **Number of intensity levels** = 2ⁿ for n bits/pixel.

### 4.4 Area-Fill Attributes
- **Fill style:** **hollow**, **solid**, **pattern**, **hatch** (diagonal/cross lines).
- **Fill color** and **pattern** (from a pixel pattern array/tile).
- **Soft (tint) fill:** fill color is blended with the existing background color (e.g., F = t·F₁ + (1−t)·B).
- **Pattern tiling:** small pattern array repeated over the area.

### 4.5 Character Attributes
- **Text attributes:** **font** (typeface), **size** (character height), **color**, **style** (bold, italic), **spacing**, **alignment** (left/center/right, top/bottom), **orientation** (text up-vector/direction).
- **Marker attributes:** marker = a single character/symbol (e.g., `*`, `+`, `o`) plotted at positions; attributes = **type, size, color**.

### 4.6 Bundled Attributes
- **Unbundled (individual) attributes:** each attribute (color, width, type) is specified separately and directly.
- **Bundled attributes:** a set of attribute values stored in a **bundle table** on each output device, selected by a single **bundle index**. Helps with **device independence** – same index gives suitable look on different devices.

---

## Quick Revision Summary

| Topic | Key Point |
|---|---|
| Raster vs Random | Pixels in frame buffer vs line commands in display file |
| CRT | Electron gun → focus → deflection → phosphor; refresh needed |
| Shadow-mask | 3 guns, used in color raster CRTs |
| DDA | Float, uses steps = max(\|dx\|,\|dy\|) |
| Bresenham | Integer; p₀ = 2Δy − Δx; p<0 → same y |
| Midpoint circle | p₀ = 1 − r; 8-way symmetry; stop at x ≥ y |
| Boundary vs Flood fill | Stops at boundary color vs replaces old color |
| Bundled attributes | Attribute set chosen via table index |

**References:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed. (Ch. 1–4); University lecture notes on Overview of Graphics Systems, Line/Circle algorithms and Attributes of Output Primitives.
