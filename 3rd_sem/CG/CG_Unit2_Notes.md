# Computer Graphics — Unit II Notes
**MCA Sem 3 | Elective A07 (MCC-2.3.4) | Ravenshaw University**
**Textbook:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed., Pearson

**Syllabus:** Two-dimensional geometric transformations (Translation, Rotation, Scaling; Matrix representation & homogeneous coordinates; Composite transformation; Reflection; Shear) → Two-dimensional viewing (Viewing pipeline; Viewing coordinate reference frame; Window-to-viewport transformation) → Clipping (Cohen-Sutherland line clipping; Sutherland-Hodgman polygon clipping)

---

## 1. Two-Dimensional Geometric Transformations

**Transformation:** operation that changes position, size or orientation of an object by changing its coordinates.
A point is written as **P = (x, y)**, new position **P′ = (x′, y′)**.

---

### 1.1 Basic Transformations

#### (a) Translation
Moves an object along a **straight-line path** to a new position by adding **translation distances (tx, ty)**.

```
x′ = x + tx
y′ = y + ty          →   P′ = P + T
```
- **Rigid-body transformation** (shape/size unchanged).
- **Example:** P(3,4), T(2,−1) → P′ = (5, 3).

#### (b) Rotation
Rotates an object about a **pivot point** along a circular path by angle **θ** (+ve θ = counter-clockwise).

**About the origin:**
```
x′ = x cosθ − y sinθ
y′ = x sinθ + y cosθ
```
**About a pivot point (xr, yr):**
```
x′ = xr + (x − xr) cosθ − (y − yr) sinθ
y′ = yr + (x − xr) sinθ + (y − yr) cosθ
```
- Rigid-body transformation.
- **Example:** P(1,0), θ = 90° about origin → x′ = 0, y′ = 1 → P′ = (0,1).

#### (c) Scaling
Changes **size** of an object by scaling factors **sx, sy**.

```
x′ = x · sx
y′ = y · sy
```
- sx, sy > 1 → enlarge; < 1 → shrink; sx = sy → **uniform** scaling; sx ≠ sy → **differential** scaling (shape distorted).
- Scaling is relative to the **origin**, so the object also moves away/towards the origin.
- **About a fixed point (xf, yf):**
```
x′ = x·sx + xf(1 − sx)
y′ = y·sy + yf(1 − sy)
```
- **Example:** P(2,3), sx = 2, sy = 3 → P′ = (4, 9).

---

### 1.2 Matrix Representation and Homogeneous Coordinates

**Matrix form (2×2 / vector):**
```
Translation:  P′ = P + T      (addition)
Rotation:     P′ = R · P      R = | cosθ  −sinθ |
                                  | sinθ   cosθ |
Scaling:      P′ = S · P      S = | sx  0 |
                                  | 0  sy |
```
**Problem:** translation uses **addition**, while rotation/scaling use **multiplication** → can't combine them into one matrix.

**Homogeneous coordinates:** represent (x, y) as **(xh, yh, h)** where x = xh/h, y = yh/h. Usually **h = 1**, so **(x, y) → (x, y, 1)**.
This lets **all transformations be written as 3×3 matrix multiplications**, so they can be **concatenated (combined)**.

```
Translation T(tx,ty)      Rotation R(θ)              Scaling S(sx,sy)
| 1  0  tx |              | cosθ  −sinθ  0 |         | sx  0   0 |
| 0  1  ty |              | sinθ   cosθ  0 |         | 0   sy  0 |
| 0  0  1  |              | 0      0     1 |         | 0   0   1 |
```
Used as **P′ = M · P** with P = [x y 1]ᵀ.

**Inverse transformations:** T⁻¹ = T(−tx, −ty); R⁻¹ = R(−θ); S⁻¹ = S(1/sx, 1/sy).

---

### 1.3 Composite Transformation
A **sequence of transformations** combined into **one matrix** by multiplying matrices. Matrices are applied **right to left** (the matrix nearest the point acts first):
**M = Mₙ · … · M₂ · M₁**

**Properties**
- Two translations: T(tx₂,ty₂)·T(tx₁,ty₁) = T(tx₁+tx₂, ty₁+ty₂) — **additive**.
- Two rotations: R(θ₂)·R(θ₁) = R(θ₁+θ₂) — **additive**.
- Two scalings: S(sx₂,sy₂)·S(sx₁,sy₁) = S(sx₁·sx₂, sy₁·sy₂) — **multiplicative**.
- Matrix multiplication is **not commutative** in general (order matters). Exceptions: translation·translation, rotation·rotation, scaling·scaling, and rotation with uniform scaling.

**Rotation about a pivot point (xr, yr)**
1. Translate pivot to origin → T(−xr, −yr)
2. Rotate about origin → R(θ)
3. Translate back → T(xr, yr)

**M = T(xr,yr) · R(θ) · T(−xr,−yr)**
```
| cosθ  −sinθ  xr(1−cosθ) + yr·sinθ |
| sinθ   cosθ  yr(1−cosθ) − xr·sinθ |
| 0      0     1                    |
```

**Scaling about a fixed point (xf, yf)**

**M = T(xf,yf) · S(sx,sy) · T(−xf,−yf)**
```
| sx  0   xf(1−sx) |
| 0   sy  yf(1−sy) |
| 0   0   1        |
```

**Worked example:** Rotate triangle A(0,0), B(1,0), C(1,1) by 90° about pivot (1,1).
cos90°=0, sin90°=1 →
x′ = 1 + (x−1)(0) − (y−1)(1) = 2 − y
y′ = 1 + (x−1)(1) + (y−1)(0) = x
→ A′ = (2,0), B′ = (2,1), C′ = (1,1).

---

### 1.4 Reflection
Produces a **mirror image**; object is rotated **180° about the reflection axis** (in 3D sense).

| Reflection about | Matrix | Effect |
|---|---|---|
| **x-axis** | `diag(1, −1, 1)` | (x, y) → (x, −y) |
| **y-axis** | `diag(−1, 1, 1)` | (x, y) → (−x, y) |
| **origin** | `diag(−1, −1, 1)` | (x, y) → (−x, −y) |
| **line y = x** | `[[0,1,0],[1,0,0],[0,0,1]]` | (x, y) → (y, x) |
| **line y = −x** | `[[0,−1,0],[−1,0,0],[0,0,1]]` | (x, y) → (−y, −x) |

**Reflection about an arbitrary line** (y = mx + c): translate line to pass through origin → rotate line onto an axis → reflect about that axis → inverse rotate → inverse translate.

---

### 1.5 Shear
Distorts the shape so that the object looks **slanted** (layers of the object slide over each other).

**X-direction shear** (shx):
```
x′ = x + shx · y          | 1  shx  0 |
y′ = y                    | 0  1    0 |
                          | 0  0    1 |
```
With reference line **y = yref**: x′ = x + shx(y − yref).

**Y-direction shear** (shy):
```
x′ = x                    | 1    0  0 |
y′ = y + shy · x          | shy  1  0 |
                          | 0    0  1 |
```
With reference line **x = xref**: y′ = y + shy(x − xref).

**Example:** Unit square A(0,0) B(1,0) C(1,1) D(0,1), x-shear shx = 2 →
A′(0,0), B′(1,0), C′(3,1), D′(2,1).

### Summary of 2D transformations
| Transformation | Changes | Rigid body? |
|---|---|---|
| Translation | Position | Yes |
| Rotation | Orientation | Yes |
| Scaling | Size | No |
| Reflection | Mirror image | Yes |
| Shear | Shape (slant) | No |

---

## 2. Two-Dimensional Viewing

**Window:** the rectangular area of the **world-coordinate** scene that we want to display (what to see).
**Viewport:** the rectangular area on the **device/screen** in which the window contents are shown (where to show it).

### 2.1 The Viewing Pipeline
Sequence of steps that maps a scene in **world coordinates** to **device coordinates**:

```
Modeling      Construct World-      Convert to       Map to Normalized     Map to Device
Coordinates → Coordinate Scene  →   Viewing      →   Viewing Coords   →    Coordinates
(MC)          (WC)                  Coordinates(VC)  (NVC) [clip here]     (DC)
```
1. **Modeling transformation:** each object built in its own coordinates is placed in the world scene.
2. **Viewing transformation:** world → **viewing coordinate system** (defines the window's position & orientation).
3. **Normalization:** viewing coords → normalized coordinates (range 0–1 or −1 to 1), making the system device independent. **Clipping** is usually done here.
4. **Workstation (viewport) transformation:** normalized → device coordinates.

Together, steps 2–4 = **window-to-viewport transformation**.

### 2.2 Viewing Coordinate Reference Frame
A **reference frame** that defines the **window's position and orientation** in the world (lets the window be rotated).

- Choose a **view reference point P₀ = (x₀, y₀)** (origin of the viewing frame, usually a window corner).
- Choose a **view-up vector V** – defines the **positive yᵥ direction**.
- Unit vectors: **v = V/|V| = (v₁, v₂)** (yᵥ axis), **u = (v₂, −v₁)** (xᵥ axis, perpendicular to v).

**World → Viewing transformation:** translate P₀ to origin, then rotate so u, v align with x, y axes:
**M_WC,VC = R · T(−x₀, −y₀)**
```
R = | u₁  u₂  0 |
    | v₁  v₂  0 |
    | 0   0   1 |
```
(If V = (0,1), no rotation is needed.)

### 2.3 Window-to-Viewport Coordinate Transformation
Maps a point **(xw, yw)** in the window to **(xv, yv)** in the viewport, keeping the **same relative position**.

Window: xwmin, xwmax, ywmin, ywmax | Viewport: xvmin, xvmax, yvmin, yvmax

Relative-position condition:
```
(xv − xvmin)/(xvmax − xvmin) = (xw − xwmin)/(xwmax − xwmin)
(yv − yvmin)/(yvmax − yvmin) = (yw − ywmin)/(ywmax − ywmin)
```
Solving:
```
xv = xvmin + (xw − xwmin) · sx      sx = (xvmax − xvmin)/(xwmax − xwmin)
yv = yvmin + (yw − ywmin) · sy      sy = (yvmax − yvmin)/(ywmax − ywmin)
```
**Steps (composite):** (1) translate window corner to origin, (2) scale by (sx, sy), (3) translate to viewport corner.

- If **sx = sy** → proportions kept; if **sx ≠ sy** → object looks stretched/squeezed.
- Changing window size → **zoom**; moving the window → **pan**.

**Example:** Window (0,0)–(100,100), Viewport (0,0)–(50,50). Point (40,60):
sx = sy = 0.5 → xv = 20, yv = 30.

---

## 3. Clipping
**Clipping:** removing the parts of a picture lying **outside** the clipping window (so only the visible part is processed/drawn). Saves time by avoiding scan conversion of invisible parts.

Types: point, line, polygon (area), curve, text clipping.

**Point clipping:** point (x, y) is displayed if
**xwmin ≤ x ≤ xwmax** and **ywmin ≤ y ≤ ywmax**.

---

### 3.1 Line Clipping — Cohen-Sutherland Algorithm
Fast method that **quickly accepts or rejects** most lines using **region (outcodes)** before computing any intersection.

**Step 1: Region codes.** Window boundaries divide the plane into **9 regions**. Each end point gets a **4-bit code**:

| Bit | Set to 1 if |
|---|---|
| Bit 1 | point is to the **Left** (x < xwmin) |
| Bit 2 | point is to the **Right** (x > xwmax) |
| Bit 3 | point is **Below** (y < ywmin) |
| Bit 4 | point is **Above** (y > ywmax) |

```
 1001 | 1000 | 1010
 -----+------+-----
 0001 | 0000 | 0010      (0000 = inside window)
 -----+------+-----
 0101 | 0100 | 0110
```

**Step 2: Tests**
| Condition | Decision |
|---|---|
| Both codes = **0000** | **Trivially accept** (fully inside) |
| **Code₁ AND Code₂ ≠ 0000** | **Trivially reject** (both on same outside side) |
| Otherwise (AND = 0000, not both 0000) | **Clip** – candidate; find intersection |

**Step 3: Clipping a candidate line**
1. Pick an endpoint whose code ≠ 0000 (outside).
2. Find the **intersection with a window boundary** indicated by a set bit, using slope m = (y₂−y₁)/(x₂−x₁):
   - Left/right boundary (x = xwmin or xwmax): **y = y₁ + m(x − x₁)**
   - Top/bottom boundary (y = ywmin or ywmax): **x = x₁ + (y − y₁)/m**
3. Replace the outside endpoint with the intersection point, recompute its code.
4. Repeat steps (tests) until the line is accepted or rejected.

**Worked example:** Window xmin=0, xmax=10, ymin=0, ymax=10. Line P₁(−5,3) → P₂(5,13).
- Codes: P₁ = 0001 (left), P₂ = 1000 (above). AND = 0000 → candidate. m = 10/10 = 1.
- Clip P₁ at x = 0: y = 3 + 1(0+5) = 8 → P₁′ = (0,8), code 0000.
- Clip P₂ at y = 10: x = 5 + (10−13)/1 = 2 → P₂′ = (2,10), code 0000.
- Both 0000 → **accept visible segment (0,8)–(2,10).**

**Advantages:** very few intersection calculations; efficient when most lines lie fully inside/outside.
**Disadvantage:** may need up to four intersection calculations for a line.

---

### 3.2 Polygon Clipping — Sutherland-Hodgman Algorithm
Clips a polygon by processing it against **each window boundary in turn** (left → right → bottom → top, any fixed order). The output vertex list of one boundary becomes input for the next.

**For each polygon edge (vertex pair S → E) against one boundary, there are 4 cases:**

| Case | S (start) | E (end) | Output vertex(es) |
|---|---|---|---|
| 1 | Inside | Inside | **E** |
| 2 | Inside | Outside | **Intersection point I** |
| 3 | Outside | Outside | **None** |
| 4 | Outside | Inside | **Intersection I, then E** |

After all four boundaries, the final vertex list = clipped polygon.

**Features**
- Works well for **convex polygons**.
- For **concave polygons** it may produce **extraneous (connecting) edges** along the window boundary when the result should be separate pieces. Fix: split the concave polygon into convex parts, or use **Weiler-Atherton** algorithm.
- Simple, easily implemented as a **pipeline** of four boundary clippers.

---

## Quick Revision Summary

| Topic | Key Point |
|---|---|
| Homogeneous coords | (x,y) → (x,y,1); all transforms become 3×3 multiplication |
| Composite order | M = Mₙ…M₂M₁ (right-to-left); not commutative |
| Pivot rotation | T(xr,yr)·R(θ)·T(−xr,−yr) |
| Fixed-pt scaling | T(xf,yf)·S·T(−xf,−yf) |
| Reflection | x-axis: diag(1,−1,1); y-axis: diag(−1,1,1); origin: diag(−1,−1,1) |
| Shear | x-shear: x′ = x + shx·y |
| Viewing pipeline | MC → WC → VC → NVC → DC |
| Window→Viewport | xv = xvmin + (xw − xwmin)·sx |
| Cohen-Sutherland | 4-bit codes; accept if both 0000; reject if AND ≠ 0000 |
| Sutherland-Hodgman | 4 cases per edge, clip boundary by boundary |

**References:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed. (Ch. 5–6); standard university lecture notes on 2D transformations, viewing and clipping.
