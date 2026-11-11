# Computer Graphics — Unit IV Notes
**MCA Sem 3 | Elective A07 (MCC-2.3.4) | Ravenshaw University**
**Textbook:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed., Pearson

**Syllabus:** Visible-surface detection methods (Classification, Back-face detection, Depth-buffer, A-buffer, Scan-line, Depth-sorting) → Illumination models (Basic illumination models, Displaying light intensities, Halftone patterns and Dithering, Polygon rendering methods: Gouraud and Phong shading)

---

## 1. Visible-Surface Detection Methods
**Visible-surface detection (hidden-surface removal):** identifying the parts of a scene that are **visible from a chosen viewing position**, so hidden parts are not displayed. Needed for realistic 3D images.

### 1.1 Classification of Visible-Surface Detection Algorithms

| | **Object-Space Methods** | **Image-Space Methods** |
|---|---|---|
| Works on | Objects/surfaces **in the scene** (compare objects with each other) | **Pixels** of the projected image (decide visible surface per pixel) |
| Accuracy | Device independent, precise | Limited to screen resolution |
| Typical cost | Grows with number of objects | Grows with number of pixels × objects |
| Examples | Back-face detection, Depth sorting (partly), BSP tree | Depth-buffer, A-buffer, Scan-line, Ray casting |

> Most practical algorithms are image-space. Depth-sorting uses **both**.

**Two main strategies:** **Sorting** (order surfaces by depth) and **Coherence** (reuse results from neighbouring areas):
- *Face coherence* – surface properties vary smoothly across a face.
- *Edge coherence* – visibility of an edge changes only where it crosses another edge.
- *Scan-line coherence* – visible spans on consecutive scan lines are similar.
- *Area coherence* – adjacent pixels usually belong to the same surface.
- *Depth coherence* – depth varies smoothly across a surface.
- *Frame coherence* – consecutive animation frames differ little.

*(Other methods in the textbook: BSP-tree, area-subdivision, octree, ray-casting, wire-frame visibility.)*

---

### 1.2 Back-Face Detection (Object-space)
Quick test to remove polygons **facing away** from the viewer.

- Plane: Ax + By + Cz + D = 0, outward normal **N = (A, B, C)**.
- Let **V** = vector along the viewing direction (from viewer toward the scene).
  - **V · N > 0** → polygon is a **back face** (remove).
  - **V · N ≤ 0** → front face (keep).
- In viewing coordinates with viewing direction along **−z_v**: V = (0,0,−1), so V·N = −C → **back face if C ≤ 0** (Hearn & Baker).
- Equivalent test: point (x,y,z) of viewer gives Ax+By+Cz+D < 0 → back face.

**Example:** V = (0,0,−1). Polygon with N = (0,0,1): V·N = −1 < 0 → front. Polygon with N = (0,0,−1): V·N = 1 > 0 → **back, discard**.

**Notes:** removes ≈ **half** the polygons; fully correct only for a **single convex polyhedron**; for concave/overlapping objects, other methods are still needed.

---

### 1.3 Depth-Buffer (Z-Buffer) Method (Image-space)
Compares **surface depths at each pixel position** (x, y) on the projection plane. Two buffers of the same resolution as the screen:
- **Depth buffer:** stores depth (z) of the nearest surface so far at each pixel.
- **Refresh (frame) buffer:** stores colour/intensity.

**Algorithm** (Hearn & Baker convention: normalized depth 0 ≤ z ≤ 1, larger z = nearer to viewer):
```
1. Initialize: depth(x,y) = 0 ; refresh(x,y) = background colour  (for all x,y)
2. For each polygon in the scene (any order):
     For each pixel (x,y) covered by the polygon:
         calculate depth z at (x,y)
         if z > depth(x,y):
             depth(x,y)   = z
             refresh(x,y) = surface colour at (x,y)
```
(Alternate convention: initialize depth to maximum, keep pixel if z is **smaller**.)

**Depth calculation from plane equation:** z = (−Ax − By − D) / C.
- **Next pixel on same scan line (x+1):** z′ = z − A/C
- **Next scan line (y−1), along an edge:** z′ = z + (A/m + B)/C (m = edge slope); for a vertical edge: z′ = z + B/C

(Incremental → only one addition per pixel.)

**Example:** pixel (10,10): surface S₁ has z = 0.3, surface S₂ has z = 0.7 → S₂ is nearer, so its colour is stored.

| Advantages | Disadvantages |
|---|---|
| Simple, no sorting needed | Extra memory (depth buffer) |
| Surfaces processed in **any order** | Only **one surface per pixel** – cannot handle transparency / antialiasing |
| Works for complex scenes, easy in hardware | Wasted work on hidden surfaces |

---

### 1.4 A-Buffer Method (Image-space)
**Antialiased, Area-averaged, Accumulation buffer** — extension of depth-buffer idea (developed at Lucasfilm). Handles **transparency** and **antialiasing** by allowing **multiple surfaces per pixel**.

Each A-buffer position holds a **linked list** of surface data:
- **Depth field:** ≥ 0 → **single surface** (stores intensity, etc.); **< 0** → **multiple surfaces** (stores pointer to a linked list).
- **Surface data fields:** RGB intensity, **opacity (transparency) parameter**, **depth**, **percent of area coverage** of the pixel, **surface identifier**, other rendering parameters, **pointer to next surface**.

**Working:** surfaces are processed (using sub-pixel area masks); depth, area coverage and opacity are used to **combine** the surface contributions into the final pixel colour.

| Advantage | Disadvantage |
|---|---|
| Transparent/translucent surfaces and antialiased edges | More memory and processing than Z-buffer |

---

### 1.5 Scan-Line Method (Image-space)
Extension of **scan-line polygon fill**; visibility is resolved **one scan line at a time**, for all polygons together.

**Data structures:** edge table, polygon table, **active edge list** (edges crossing current scan line, sorted by x), and an **in/out flag** for each polygon.

**Procedure for each scan line:**
1. Update the active edge list; for each pair of edges, flip the polygon flag **on/off** as the line crosses into/out of the polygon.
2. For each **span** (interval between edge intersections):
   - **No** flag on → background colour.
   - **One** flag on → intensity of that surface.
   - **More than one** flag on → compute **depth** of each overlapping surface (using plane equation) at the span; the **nearest** surface is visible.
3. Move to next scan line, using **coherence** (edge intersection x increments by 1/m, depth increments are constant).

**Notes:** depth calculations are needed only where surfaces **overlap in a span**. Intersecting/cyclically overlapping surfaces must be **split** at the intersection (or the method may fail).

| Advantage | Disadvantage |
|---|---|
| Needs less memory than Z-buffer (one scan line at a time) | More complex data structures |
| Efficient with coherence | Problems with intersecting/cyclic surfaces |

---

### 1.6 Depth-Sorting Method (Painter's Algorithm)
Uses **both** image-space (scan conversion) and object-space (sorting) operations. Like a painter, **paint distant surfaces first**, nearer ones paint over them.

**Steps**
1. **Sort** all surfaces by **decreasing depth** (farthest z first; key = farthest point z_min).
2. **Resolve ambiguities** where surfaces overlap in depth.
3. **Scan convert** surfaces in order, **back to front**.

**Overlap test** — for surface S (farthest) with an overlapping surface S′, S can be scan-converted **if any** of these holds (tests in order of difficulty):
1. Their **bounding rectangles in the xy-plane don't overlap**.
2. S is **completely behind** S′ (relative to the viewer).
3. S′ is **completely in front** of S.
4. Their **projections onto the view plane don't overlap**.

If **all fail**, **swap** the order of S and S′ and retest; if still failing (cyclic overlap/intersection), **split** one surface along the other's plane and repeat.

| Advantage | Disadvantage |
|---|---|
| Simple idea; handles overlapping surfaces without per-pixel depth | Sorting overhead; special cases (cycles, intersection) need splitting |
| Works well for few polygons | Can paint the same pixel several times (overdraw) |

### Comparison summary
| Method | Space | Handles transparency | Memory | Idea |
|---|---|---|---|---|
| Back-face | Object | No | Low | N·V test |
| Depth-buffer | Image | No | High | Nearest z per pixel |
| A-buffer | Image | **Yes** | Higher | Linked list of surfaces per pixel |
| Scan-line | Image | Limited | Low | Spans on each scan line |
| Depth-sorting | Both | Limited | Low | Back-to-front painting |

---

## 2. Illumination Models
**Illumination (lighting) model:** equation(s) that calculate the **light intensity at a surface point** from light sources and surface properties. **Surface rendering** uses it to shade polygons.

**Light sources:** *Point source* (rays diverge from a point), *Distributed (extended) source*. Surface properties: colour, reflectivity, transparency, texture.

### 2.1 Basic Illumination Models

#### (a) Ambient Light
Uniform background light from surroundings (non-directional); gives every surface a basic brightness.
**I_ambient = kₐ · Iₐ**  (kₐ = ambient-reflection coefficient, 0–1; Iₐ = ambient intensity)

#### (b) Diffuse Reflection
Light scattered **equally in all directions** by a dull/matte surface; brightness independent of viewer position. Follows **Lambert's cosine law**:
**I_diff = k_d · I_l · cosθ = k_d · I_l · (N · L)**, 0° ≤ θ ≤ 90°
- N = unit surface normal, L = unit vector from point to light source, θ = angle between N and L.
- With ambient: **I = kₐIₐ + k_d I_l (N·L)**.

#### (c) Specular Reflection and the Phong Model
Bright **highlight** on shiny surfaces; viewer-dependent. **Phong specular model:**
**I_spec = k_s · I_l · cosⁿˢφ = k_s · I_l · (V · R)ⁿˢ**
- R = unit reflection vector = **2(N·L)N − L**; V = unit vector toward viewer; φ = angle between V and R.
- **n_s = specular-reflection exponent** (shininess): 1 = rough, large (100+) = near-perfect mirror (narrow highlight).
- k_s = specular coefficient.

**Blinn–Phong simplification:** halfway vector **H = (L + V)/|L + V|**; use **(N · H)ⁿˢ** instead of (V·R)ⁿˢ (cheaper).

#### Combined model (multiple light sources)
**I = kₐIₐ + Σᵢ I_lᵢ [ k_d (N·Lᵢ) + k_s (N·Hᵢ)ⁿˢ ]**

**Example:** kₐ = 0.1, Iₐ = 1, k_d = 0.6, I_l = 1, N·L = 0.5, k_s = 0.3, N·H = 0.8, n_s = 10
I = 0.1 + 0.6(0.5) + 0.3(0.8¹⁰) = 0.1 + 0.3 + 0.3(0.107) ≈ **0.432**

**Other factors (brief)**
- **Intensity attenuation:** light weakens with distance d: **f(d) = 1/(a₀ + a₁d + a₂d²)** (inverse-square for point source).
- **Colour considerations:** apply the model separately to R, G, B.
- **Transparency** and **shadows** can be added to the basic model.

---

### 2.2 Displaying Light Intensities
Calculated intensities (0.0–1.0) must be mapped to the **limited intensity levels** the display provides.

**Assigning intensity levels**
- Human eye perceives **intensity ratios** (logarithmic), not equal steps. For **n** levels use **Iₖ = r · Iₖ₋₁ = rᵏ I₀**, with a constant ratio r (~1.01 for smooth appearance), so levels are spaced **logarithmically**.

**Gamma correction and video look-up tables**
- CRT brightness is **non-linear** with input voltage: **I = k · Vᵞ**.
- **Gamma correction:** apply inverse **V = (I/k)^(1/γ)** (γ ≈ 2.2 typically) so displayed intensity matches the calculated one.
- Done using a **video look-up table (LUT)** that stores corrected values.

---

### 2.3 Halftone Patterns and Dithering Techniques
For **bilevel devices** (only on/off pixels, e.g., printers) – **continuous-tone images** are approximated by exploiting the eye's spatial integration.

#### Halftone Approximation
- Newspaper style: different-size **dots** represent intensity.
- On raster: group pixels into an **n × n pattern**; number of ON pixels gives intensity. **n × n block → n² + 1 intensity levels.**
  - 2 × 2 → **5 levels** (0–4 pixels on); 3 × 3 → 10 levels.
- **Trade-off:** more intensity levels ⇒ **lower spatial resolution** (resolution becomes 1/n).
- Pattern rules: avoid straight-line patterns, "grow" pixels from the previous level (pixel on at level k stays on at level k+1).

#### Dithering Techniques
Approximate halftones **without losing resolution** (each pixel set independently).

1. **Ordered dither (dither matrix):** pixel (x, y) is turned **on** if intensity **I > Dₙ[x mod n, y mod n]**.
```
D2 = | 3 1 |        D4 = | 15  7 13  5 |
     | 0 2 |             |  3 11  1  9 |
                          | 12  4 14  6 |
                          |  0  8  2 10 |
```
   Higher matrix built **recursively** from D₂: Dₙ = [[4Dₙ/₂ + 3U, 4Dₙ/₂ + 1U],[4Dₙ/₂ + 0U, 4Dₙ/₂ + 2U]] (U = all-ones matrix). Image intensities are scaled to 0 … n²−1.

2. **Error diffusion (Floyd–Steinberg):** compare pixel to threshold, set on/off, and **distribute the error** to neighbours not yet processed:
   - **7/16** → right pixel, **3/16** → below-left, **5/16** → below, **1/16** → below-right.
   Gives better quality than ordered dither.

3. **Random dither:** add random noise before thresholding (simple but noisy).

---

### 2.4 Polygon-Rendering (Surface Rendering) Methods
Apply the illumination model to a polygon mesh to give **shaded** appearance.

#### (a) Constant-Intensity (Flat) Shading
- **One intensity per polygon** computed using the polygon normal.
- Fast; fine for **small** polygons with distant light/viewer.
- Disadvantage: **faceted look**, **Mach-band** effect (eye exaggerates intensity differences at edges).

#### (b) Gouraud Shading (Intensity Interpolation)
Interpolates **intensities** across polygon.

**Steps**
1. **Vertex normal** = average of normals of all polygons sharing the vertex: **N_V = Σ Nₖ / |Σ Nₖ|**.
2. Apply illumination model at each **vertex** → vertex intensities.
3. **Linearly interpolate along edges** (scan line y between vertices 1 and 2):
   **I_a = (y_a − y₂)/(y₁ − y₂) · I₁ + (y₁ − y_a)/(y₁ − y₂) · I₂**
4. **Interpolate along the scan line** between edge points a and b:
   **I_p = (x_b − x_p)/(x_b − x_a) · I_a + (x_p − x_a)/(x_b − x_a) · I_b**
   (incrementally: I(x+1) = I(x) + ΔI).

**Example:** I₁ = 0.8 at y = 10, I₂ = 0.4 at y = 20; at y = 15: I_a = 0.5(0.8) + 0.5(0.4) = **0.6**.

| Advantages | Disadvantages |
|---|---|
| Smooth shading, removes facets; faster than Phong | **Highlights** may be missed/distorted (intensity only computed at vertices) |
| Easy incremental computation | Mach bands may still appear |

#### (c) Phong Shading (Normal-Vector Interpolation)
Interpolates the **surface normals**, and applies the illumination model **at every pixel**.

**Steps**
1. Compute (averaged) **vertex normals** as in Gouraud.
2. Interpolate normals along edges: **N = (y − y₂)/(y₁ − y₂) · N₁ + (y₁ − y)/(y₁ − y₂) · N₂**.
3. Interpolate along the scan line between edge normals.
4. **Normalize** the normal at each pixel and apply the **illumination model** per pixel.

| Advantages | Disadvantages |
|---|---|
| Accurate **specular highlights**, smoother, fewer Mach bands | **Much slower** (per-pixel lighting + normalization) |

**Fast Phong shading:** approximates the normalization using a **Taylor-series (incremental) expansion** of the interpolated normal → reduces computation.

#### Gouraud vs Phong
| Basis | Gouraud | Phong |
|---|---|---|
| Interpolates | **Intensity** | **Normal vector** |
| Lighting computed | At vertices only | At **every pixel** |
| Highlights | May be lost | Accurate |
| Speed | Faster | Slower |
| Quality | Good | Better |

---

## Quick Revision Summary

| Topic | Key Point |
|---|---|
| Object vs Image space | Compare objects vs compare pixels |
| Back-face | V·N > 0 → back (C ≤ 0 in viewing coords) |
| Depth-buffer | z-compare per pixel; z′ = z − A/C along a scan line |
| A-buffer | Linked list per pixel; transparency + antialiasing |
| Scan-line | In/out flags; compare depth only in overlapping spans |
| Depth-sorting | Back-to-front; 4 overlap tests, then swap/split |
| Ambient / Diffuse | kₐIₐ ; k_d I_l (N·L) (Lambert) |
| Specular (Phong) | k_s I_l (V·R)ⁿˢ ; R = 2(N·L)N − L |
| Halftone | n×n block → n²+1 levels, resolution lost |
| Dither | I > Dₙ[x mod n, y mod n]; D2 = [[3,1],[0,2]] |
| Error diffusion | 7/16, 3/16, 5/16, 1/16 |
| Gouraud vs Phong | Interpolate intensity vs interpolate normals |

**References:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed. (Ch. 13–14); standard university lecture notes on visible-surface detection, illumination models and surface rendering.
