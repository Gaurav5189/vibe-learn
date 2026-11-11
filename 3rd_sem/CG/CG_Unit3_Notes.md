# Computer Graphics — Unit III Notes
**MCA Sem 3 | Elective A07 (MCC-2.3.4) | Ravenshaw University**
**Textbook:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed., Pearson

**Syllabus:** Three-dimensional object representation (Polygon surfaces, Quadric surfaces, Spline representations, Bezier curves and surfaces, B-spline curves and surfaces) → Three-dimensional geometric and modeling transformations (Translation, Rotation, Scaling, Reflections, Shears, Composite transformation) → Three-dimensional viewing (Viewing pipeline, Viewing coordinates, Projections: Parallel and Perspective)

---

## 1. Three-Dimensional Object Representation

Different objects (buildings, terrain, clouds, cars) need different representations. Two broad categories:

| Category | Idea | Examples |
|---|---|---|
| **Boundary Representation (B-rep)** | Object = set of **surfaces** separating interior from outside | Polygon facets, spline patches |
| **Space-partitioning representation** | Object **interior** divided into small non-overlapping solids | Octree |

Other methods: procedural (fractals, particle systems), physically based models, sweep representations, CSG (constructive solid geometry).

---

### 1.1 Polygon Surfaces
Most common B-rep: object surface = **set of polygons** (polyhedron). Curved surfaces are approximated by many small polygons (**tessellation**). Fast to render, hardware-friendly.

#### Polygon Tables
Data of a polygon surface is stored in tables:
1. **Geometric table:** *Vertex table* (coordinates), *Edge table* (vertex pairs), *Surface/polygon table* (edges forming each polygon).
2. **Attribute table:** color, transparency, texture, reflectivity etc.

**Consistency checks:** every vertex is an endpoint of ≥ 2 edges; every edge belongs to ≥ 1 polygon; every polygon is closed.

#### Plane Equation
Each polygon lies in a plane:

**Ax + By + Cz + D = 0**

- (A, B, C) = **surface normal N**. Found by solving for 3 non-collinear polygon vertices, or **N = (V₂ − V₁) × (V₃ − V₁)** (vertices in counter-clockwise order when seen from outside).
- Using vertices (x₁,y₁,z₁), (x₂,y₂,z₂), (x₃,y₃,z₃):
```
A = y₁(z₂ − z₃) + y₂(z₃ − z₁) + y₃(z₁ − z₂)
B = z₁(x₂ − x₃) + z₂(x₃ − x₁) + z₃(x₁ − x₂)
C = x₁(y₂ − y₃) + x₂(y₃ − y₁) + x₃(y₁ − y₂)
D = −x₁(y₂z₃ − y₃z₂) − x₂(y₃z₁ − y₁z₃) − x₃(y₁z₂ − y₂z₁)
```
- **Point test** for (x, y, z): 
  - Ax + By + Cz + D **< 0** → point **inside** (behind the plane)
  - **= 0** → on the plane
  - **> 0** → **outside** (in front)
  (valid when the normal points outward; used in back-face detection).

#### Polygon Meshes
Many polygons joined edge-to-edge to model a surface.
- **Triangle strip:** n−2 triangles from n vertices.
- **Quadrilateral mesh:** n × m vertices → (n−1)(m−1) quadrilaterals.
- **Euler's formula (simple polyhedron):** **V − E + F = 2**.
- **Wire-frame model:** only vertices and edges displayed — simple, easy to clip, but not realistic.

---

### 1.2 Quadric Surfaces
Surfaces described by **second-degree equations** (quadratics). Common in engineering models.

| Surface | Equation | Parametric form |
|---|---|---|
| **Sphere** (radius r, centre at origin) | x² + y² + z² = r² | x = r cosφ cosθ, y = r cosφ sinθ, z = r sinφ  (−π/2 ≤ φ ≤ π/2, −π ≤ θ ≤ π) |
| **Ellipsoid** (radii rₓ, r_y, r_z) | (x/rₓ)² + (y/r_y)² + (z/r_z)² = 1 | x = rₓ cosφ cosθ, y = r_y cosφ sinθ, z = r_z sinφ |
| **Torus** (doughnut) | [ r − √((x/rₓ)² + (y/r_y)²) ]² + (z/r_z)² = 1 | x = rₓ(r + cosφ)cosθ, y = r_y(r + cosφ)sinθ, z = r_z sinφ  (−π ≤ φ,θ ≤ π) |

Here φ = latitude angle, θ = longitude angle. (Also: *superquadrics* add exponents s₁, s₂ for more shapes; *blobby objects* = shapes that merge/blend like molecules or water drops.)

---

### 1.3 Spline Representations
**Spline:** originally a flexible strip used by draftsmen to draw smooth curves through points. In graphics: a **piecewise polynomial curve** (usually **cubic**) with smooth joins between sections.

- **Control points:** points that define the curve.
- **Interpolation spline:** curve **passes through** all control points.
- **Approximation spline:** curve is **guided by** control points but need not pass through them (Bezier, B-spline).
- **Control graph / control polygon:** polyline joining control points in order.
- **Convex hull:** smallest convex polygon containing all control points; curve lies inside it.

**Cubic parametric form** (0 ≤ u ≤ 1):
```
x(u) = aₓu³ + bₓu² + cₓu + dₓ
y(u) = a_y u³ + b_y u² + c_y u + d_y
z(u) = a_z u³ + b_z u² + c_z u + d_z
```
In matrix form: **P(u) = U · M_spline · M_geometry**, where U = [u³ u² u 1], M_geometry = control-point/constraint matrix, M_spline = basis (blending) matrix.

**Continuity at joints of curve sections**
| Parametric | Meaning | Geometric | Meaning |
|---|---|---|---|
| **C⁰** | Sections just meet (same point) | **G⁰** | Same as C⁰ |
| **C¹** | Same **first derivative** (tangent vector) at joint | **G¹** | Tangents have same **direction** (magnitudes may differ) |
| **C²** | Same first **and second** derivatives | **G²** | Same curvature |

Higher continuity → smoother curve. Examples of splines: natural cubic, Hermite, Cardinal, Kochanek-Bartels, Bezier, B-spline.

---

### 1.4 Bezier Curves and Surfaces
Developed by **Pierre Bezier** (Renault) for car-body design. An **approximation spline**.

#### Bezier Curve
Given **n + 1 control points** p₀ … pₙ, the curve is

**P(u) = Σₖ₌₀ⁿ pₖ · BEZₖ,ₙ(u),  0 ≤ u ≤ 1**

**Bernstein blending functions:**
**BEZₖ,ₙ(u) = C(n,k) · uᵏ · (1 − u)ⁿ⁻ᵏ**, where C(n,k) = n! / (k!(n−k)!)

**Common cases**
- **Quadratic (n = 2):** (1−u)², 2u(1−u), u²
- **Cubic (n = 3):** BEZ₀ = (1−u)³, BEZ₁ = 3u(1−u)², BEZ₂ = 3u²(1−u), BEZ₃ = u³

**Cubic matrix form:** P(u) = [u³ u² u 1] · M_Bez · [p₀ p₁ p₂ p₃]ᵀ
```
M_Bez = | −1   3  −3  1 |
        |  3  −6   3  0 |
        | −3   3   0  0 |
        |  1   0   0  0 |
```

**Properties**
1. Degree = **number of control points − 1**.
2. Curve passes through the **first and last** control points: P(0) = p₀, P(1) = pₙ.
3. **Tangent** at start is along p₀p₁; at end along pₙ₋₁pₙ.
4. Curve lies within the **convex hull** of control points.
5. Blending functions are **positive** and their **sum = 1** (so the curve is a weighted average of control points).
6. **Global control:** moving any one control point changes the **whole** curve.
7. Can be **subdivided** at any u into two Bezier curves.
8. Closed curve: choose p₀ = pₙ. Repeating a point pulls the curve closer to it.
9. **Continuity between two sections:** C⁰ if p₀′ = pₙ; C¹ if p₁′ − p₀′ = pₙ − pₙ₋₁ (collinear + equal spacing).

**Example:** Cubic, p₀(0,0), p₁(1,2), p₂(3,2), p₃(4,0), at u = 0.5. Weights = 1/8, 3/8, 3/8, 1/8
x = 0 + 3/8 + 9/8 + 4/8 = **2**; y = 0 + 6/8 + 6/8 + 0 = **1.5** → **P(0.5) = (2, 1.5)**.

**Limitations:** degree rises with the number of points; no local control.

#### Bezier Surface
Cartesian product of two sets of Bezier curves (control net of (m+1) × (n+1) points):

**P(u,v) = Σⱼ₌₀ᵐ Σₖ₌₀ⁿ pⱼ,ₖ · BEZⱼ,ₘ(v) · BEZₖ,ₙ(u),  0 ≤ u, v ≤ 1**

- **Bicubic Bezier patch:** m = n = 3 → **16 control points**.
- Same properties: surface passes through corner control points, lies in convex hull, boundary curves are Bezier curves.
- Adjacent patches joined by matching boundary control points (continuity conditions).

---

### 1.5 B-Spline Curves and Surfaces
Removes Bezier limitations: gives **local control** and degree **independent** of the number of control points.

#### B-Spline Curve
**P(u) = Σₖ₌₀ⁿ pₖ · Bₖ,d(u),  u_min ≤ u ≤ u_max,  2 ≤ d ≤ n + 1**

- n + 1 control points; **d = order** (degree = d − 1; d=4 gives cubic).
- **Knot vector:** non-decreasing values u₀ ≤ u₁ … with **n + d + 1** knots.

**Cox–de Boor recursion (blending functions):**
```
Bₖ,₁(u) = 1   if uₖ ≤ u < uₖ₊₁,   0 otherwise
Bₖ,d(u) = (u − uₖ)/(uₖ₊d₋₁ − uₖ) · Bₖ,d₋₁(u) + (uₖ₊d − u)/(uₖ₊d − uₖ₊₁) · Bₖ₊₁,d₋₁(u)
```

**Types (by knot vector)**
| Type | Knot vector | Behaviour |
|---|---|---|
| **Uniform (periodic)** | Equal spacing, e.g. {0,1,2,3,4,5,6} (d=3, n=3) | Same blending shape for each section; curve does **not** touch end points |
| **Open uniform** | First and last knot repeated **d** times, e.g. {0,0,0,1,2,2,2} | Curve **passes through first and last** control points (like Bezier) |
| **Non-uniform** | Unequal spacing / repeated interior knots | More flexible shape control |

**Cubic periodic B-spline matrix:**
```
M_B = 1/6 | −1   3  −3  1 |
          |  3  −6   3  0 |
          | −3   0   3  0 |
          |  1   4   1  0 |
```

**Properties**
1. Local control — a control point affects only a few spans (where its Bₖ,d ≠ 0).
2. Degree independent of number of control points.
3. Blending functions ≥ 0 and sum to 1 → curve in convex hull (tighter than Bezier).
4. Variation-diminishing property; C^(d−2) continuity at knots (cubic → C²).
5. **Bezier curve = special case** of B-spline (d = n + 1, open uniform knots).

#### B-Spline Surface
**P(u,v) = Σⱼ₌₀ⁿ¹ Σₖ₌₀ⁿ² pⱼ,ₖ · Bⱼ,d₁(v) · Bₖ,d₂(u)** — tensor product of two B-spline curves; local control over patches.

#### Bezier vs B-Spline
| Feature | Bezier | B-Spline |
|---|---|---|
| Degree | n (points − 1) | Chosen (d − 1), independent of points |
| Control | Global | Local |
| Blending function | Bernstein polynomial | Cox–de Boor (piecewise) |
| Knot vector | Not needed | Required |
| Passes through end points | Always | Only for open knot vector |
| Complexity | Simpler | More flexible |

---

## 2. Three-Dimensional Geometric and Modeling Transformations
Extension of 2D: points are (x, y, z); using **homogeneous coordinates (x, y, z, 1)** all transformations are **4 × 4 matrices**, concatenated by multiplication (**right-handed** coordinate system; positive rotation = counter-clockwise looking from positive axis towards origin).

### 2.1 Translation
```
x′ = x + tx,  y′ = y + ty,  z′ = z + tz            T(tx,ty,tz) = | 1 0 0 tx |
                                                                  | 0 1 0 ty |
                                                                  | 0 0 1 tz |
                                                                  | 0 0 0 1  |
```
Inverse: T(−tx, −ty, −tz). **Example:** (1,2,3), T(2,−1,4) → **(3,1,7)**.

### 2.2 Rotation
**About z-axis (angle θ):** x′ = x cosθ − y sinθ, y′ = x sinθ + y cosθ, z′ = z
**About x-axis:** y′ = y cosθ − z sinθ, z′ = y sinθ + z cosθ, x′ = x
**About y-axis:** z′ = z cosθ − x sinθ, x′ = z sinθ + x cosθ, y′ = y
(Cyclic order x → y → z → x.)

```
Rz(θ) = | cosθ −sinθ 0 0 |   Rx(θ) = | 1  0     0     0 |   Ry(θ) = |  cosθ 0 sinθ 0 |
        | sinθ  cosθ 0 0 |           | 0  cosθ −sinθ  0 |           |  0    1 0    0 |
        | 0     0    1 0 |           | 0  sinθ  cosθ  0 |           | −sinθ 0 cosθ 0 |
        | 0     0    0 1 |           | 0  0     0     1 |           |  0    0 0    1 |
```
Inverse: R(−θ) = transpose of R(θ).

**Rotation about an axis parallel to a coordinate axis:** (1) translate axis to coincide with the coordinate axis, (2) rotate, (3) translate back → **T⁻¹ · R(θ) · T**.

**Rotation about an arbitrary axis** (through P₁, direction vector):
1. Translate P₁ to origin.
2. Rotate about x-axis, then y-axis, to align axis with **z-axis**.
3. Rotate by θ about z-axis.
4. Apply inverse rotations (y, then x).
5. Translate back.
**R(θ) = T⁻¹ · Rx⁻¹ · Ry⁻¹ · Rz(θ) · Ry · Rx · T**

**Example:** (1,0,0) rotated 90° about z → **(0,1,0)**.

### 2.3 Scaling
```
x′ = x·sx, y′ = y·sy, z′ = z·sz            S(sx,sy,sz) = | sx 0  0  0 |
                                                          | 0  sy 0  0 |
                                                          | 0  0  sz 0 |
                                                          | 0  0  0  1 |
```
- Relative to **origin**; uniform if sx = sy = sz.
- **About a fixed point (xf,yf,zf):** **T(xf,yf,zf) · S(sx,sy,sz) · T(−xf,−yf,−zf)**
- Inverse: S(1/sx, 1/sy, 1/sz). **Example:** (1,2,3), S(2,2,2) → **(2,4,6)**.

### 2.4 Reflection
Mirror image about a **plane**:
| Reflection about | Matrix | Effect |
|---|---|---|
| **xy-plane** | diag(1, 1, −1, 1) | z → −z |
| **yz-plane** | diag(−1, 1, 1, 1) | x → −x |
| **xz-plane** | diag(1, −1, 1, 1) | y → −y |

### 2.5 Shear
Distorts shape. **z-axis shear** (x, y changed by z):
```
x′ = x + a·z,  y′ = y + b·z,  z′ = z          SHz = | 1 0 a 0 |
                                                      | 0 1 b 0 |
                                                      | 0 0 1 0 |
                                                      | 0 0 0 1 |
```
With reference position zref: x′ = x + a(z − zref), y′ = y + b(z − zref). x-axis shear and y-axis shear are defined similarly.

### 2.6 Composite Transformation
Concatenate matrices: **M = Mₙ · … · M₂ · M₁** (applied **right to left**). Order matters (not commutative).
**Example (scale about fixed point then translate):** M = T(tx,ty,tz) · T(xf,yf,zf) · S · T(−xf,−yf,−zf).

### 2.7 Modeling and Coordinate Transformations
- **Modeling transformation:** places each object (defined in its own **modeling coordinates**) into the **world coordinate** scene using T, R, S.
- **Coordinate transformation:** re-expresses the same point in a different coordinate system: translate new origin to old origin, then rotate axes to align → **M = R · T(−x₀, −y₀, −z₀)**.

---

## 3. Three-Dimensional Viewing
Unlike 2D, 3D needs a **projection** (3D → 2D view plane) and more choices: viewing position, direction, view-plane, and clipping volume.

### 3.1 Viewing Pipeline
```
Modeling      World        Viewing        Projection      Normalized        Device
Coords   →    Coords   →   Coords    →    Coords     →    Projection   →    Coords
        (modeling   (viewing     (projection    Coords       (workstation
         transf.)    transf.)     transf.)      [clipping]    transf.)
```
1. Modeling transformation → world scene.
2. **Viewing transformation** → viewing (camera) coordinates.
3. **Projection transformation** → 2D coordinates on view plane.
4. Normalization and **clipping** against the view volume.
5. **Workstation transformation** → device (screen) coordinates.

### 3.2 Viewing Coordinates
Defines a **camera/viewer** frame in the world.

**Specification**
- **View reference point P₀ = (x₀,y₀,z₀):** origin of viewing system (point being looked at / on the view plane).
- **View-plane normal vector N:** defines the view plane direction (plane ⟂ N); gives the **z_v axis**.
- **View-up vector V:** gives the **y_v** direction (projected onto the view plane).
- **View distance:** position of the view plane along z_v.

**Axes (unit vectors)**
```
n = N / |N|            (z_v axis)
u = (V × n) / |V × n|  (x_v axis)
v = n × u              (y_v axis)
```
**World → viewing transformation:** **M_WC,VC = R · T(−x₀, −y₀, −z₀)**
```
R = | u₁ u₂ u₃ 0 |
    | v₁ v₂ v₃ 0 |
    | n₁ n₂ n₃ 0 |
    | 0  0  0  1 |
```
(Steps: translate P₀ to origin, then rotate so u, v, n align with x, y, z.)

### 3.3 Projections
**Projection:** transforms 3D object points to the 2D **view plane (projection plane)**, along **projectors** (projection lines) from the **center of projection**.

#### (A) Parallel Projection
- Center of projection at **infinity** → projectors are **parallel**.
- **Parallel lines remain parallel**, relative proportions/true measurements preserved; no realistic depth (no foreshortening by distance).
- Used in engineering/architectural drawings.

**Types**
1. **Orthographic** (projectors ⟂ view plane): x_p = x, y_p = y.
   - **Multiview (front, side, top):** plane ⟂ a principal axis.
   - **Axonometric:** plane not ⟂ any principal axis — **isometric** (equal foreshortening on 3 axes), **dimetric** (2 equal), **trimetric** (all different).
2. **Oblique** (projectors **not** ⟂ view plane): shows front face true shape plus depth.
   - x_p = x + z·(L₁ cosφ), y_p = y + z·(L₁ sinφ), where **L₁ = 1/tanα**
   - **Cavalier:** α = 45° (tanα = 1) → L₁ = 1; **depth lines keep true length** (looks too long).
   - **Cabinet:** α ≈ 63.4° (tanα = 2) → L₁ = ½; **depth lines half length** (more realistic).

#### (B) Perspective Projection
- Projectors **converge** at the **center (point) of projection**; like human eye / camera.
- **Perspective foreshortening:** farther objects look smaller; parallel lines (not parallel to view plane) **converge to vanishing points**.
- Gives realistic view but **does not preserve** sizes/angles.

**Equations** (view plane at z = z_vp, projection reference point at (0,0,z_prp), d_p = z_prp − z_vp):
```
x_p = x · d_p / (z_prp − z)
y_p = y · d_p / (z_prp − z)
```
Simple case — center of projection at origin, view plane z = d:  **x_p = x·d / z,  y_p = y·d / z**.
**Example:** d = 2, point (4,6,8) → x_p = 4·2/8 = **1**, y_p = 6·2/8 = **1.5**.

**Vanishing points:** where projections of parallel lines meet. **Principal vanishing points** — lines parallel to the principal axes.
| Type | Principal axes cut by view plane | Vanishing points |
|---|---|---|
| **One-point** | 1 (plane ⟂ one axis) | 1 |
| **Two-point** | 2 | 2 |
| **Three-point** | 3 | 3 |

**View volume:** parallel projection → **rectangular parallelepiped**; perspective → **frustum** (truncated pyramid). Objects outside are clipped.

#### Parallel vs Perspective
| Basis | Parallel | Perspective |
|---|---|---|
| Center of projection | At infinity | Finite point |
| Projectors | Parallel | Converge |
| Size vs distance | Constant | Smaller when farther |
| Parallel lines | Stay parallel | Converge (vanishing point) |
| Shape/measure preserved | Yes | No |
| Realism | Low | High |
| Use | CAD, engineering drawings | Games, animation, architecture views |

---

## Quick Revision Summary

| Topic | Key Point |
|---|---|
| Plane equation | Ax+By+Cz+D=0; N=(A,B,C); <0 inside, >0 outside |
| Euler's formula | V − E + F = 2 |
| Sphere | x=r cosφ cosθ, y=r cosφ sinθ, z=r sinφ |
| Continuity | C⁰ meet; C¹ same tangent; C² same curvature |
| Bezier | Bernstein blending; degree = pts−1; global control; passes through ends |
| Bicubic Bezier patch | 16 control points |
| B-spline | Cox–de Boor; local control; knots = n+d+1 |
| 3D matrices | 4×4 homogeneous; M = Mₙ…M₁ |
| Rotation about arbitrary axis | T⁻¹·Rx⁻¹·Ry⁻¹·Rz·Ry·Rx·T |
| Viewing transform | M = R·T(−P₀); n=N/\|N\|, u=V×n, v=n×u |
| Parallel | Orthographic / oblique (cavalier α=45°, cabinet tanα=2) |
| Perspective | x_p = x·d/z; 1-, 2-, 3-point vanishing points |

**References:** Hearn & Baker, *Computer Graphics C Version*, 2nd Ed. (Ch. 9–12); standard university lecture notes on 3D object representation, transformations and viewing.
