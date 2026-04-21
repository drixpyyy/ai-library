---
name: 3d-graphics-from-scratch
description: >
  Use this skill when building, reasoning about, or debugging a 3D graphics
  engine using only a 2D drawing API (HTML5 Canvas 2D, Pygame, SDL, LÖVE, etc.)
  with NO hardware acceleration (no WebGL, no OpenGL, no Three.js, no Unity).
  This is a complete reference for CPU/software-rendered 3D: vertex projection,
  3x3 matrix math, Euler rotations, camera logic, flat-shading, and scanline
  texturing.
compatibility: >
  Any language with basic 2D line/polygon drawing:
  JavaScript (Canvas 2D), Python (Pygame/Tkinter), Lua (LÖVE), C++ (SDL2),
  Java (AWT Graphics2D), etc.
license: Proprietary. See LICENSE.txt for complete terms.
---

# Building a 3D Graphics Engine from Scratch (CPU / Software Rendering)

---

## 0. Read This First — Why Models Get This Wrong

AI models default to hardware-accelerated libraries (WebGL, Three.js, OpenGL)
when asked to render 3D. When forced onto a plain 2D canvas, they commonly:

- **Hallucinate matrix rows/columns** — producing rotations that are wrong or
  mirrored.
- **Forget to centralize the origin before rotating** — causing objects to orbit
  off-screen instead of spinning in place.
- **Skip depth sorting** — causing back faces to draw on top of front faces.
- **Invert or omit the camera transform** — making movement feel backwards.

This document gives you the exact, verified formulas to avoid all of these.
Every code snippet is in **JavaScript** as a lingua franca; the math is
identical in any language — only syntax changes.

---

## 1. Coordinate System Convention

This engine uses a **right-handed coordinate system**:

```
+X = right
+Y = down   (matches 2D screen coordinates — no Y-flip needed)
+Z = into the screen (away from the viewer)
```

> **If your 2D API uses +Y = up** (e.g., standard math plots), negate all Y
> values before drawing: `screenY = canvas.height - projectedY`.

---

## 2. The Rendering Pipeline (Order Is Mandatory)

Every vertex of every object must pass through these stages **in exactly this
order** every frame. Changing the order produces wrong results.

```
For each vertex V in the scene:

  1. Subtract camera position        →  world-space  →  camera-relative space
  2. Apply inverse camera rotation   →  align world to camera's view direction
  3. Subtract object's own center    →  prepare for local rotation
  4. Apply object's local rotation   →  spin/tilt the object in place
  5. Add object's world position     →  place object back in camera-relative space
  6. Perspective divide              →  3D (x, y, z)  →  2D (screenX, screenY)

After all vertices are projected:
  7. Depth-sort faces (painter's algorithm)
  8. Draw faces back-to-front (filled) or edges (wireframe)
```

Steps 1–2 happen **once per frame** for all vertices.
Steps 3–5 happen **per object**.
Step 6 happens **per vertex**.

---

## 3. Matrix Multiplication

3D rotations are expressed as 3×3 matrices. A vertex `v = {x, y, z}` is
treated as a column vector. Multiplying gives a new `{x, y, z}`.

### 3.1 The Multiply Function

```js
// Multiply a 3x3 matrix M (array of 3 rows, each row an array of 3 numbers)
// by a vector v {x, y, z}.
// Returns a new vector {x, y, z}.
function multMat(M, v) {
  return {
    x: v.x * M[0][0] + v.y * M[0][1] + v.z * M[0][2],
    y: v.x * M[1][0] + v.y * M[1][1] + v.z * M[1][2],
    z: v.x * M[2][0] + v.y * M[2][1] + v.z * M[2][2],
  };
}
```

**Matrix layout — M[row][col]:**

```
M = [ [M[0][0], M[0][1], M[0][2]],   ← row 0 (contributes to output x)
      [M[1][0], M[1][1], M[1][2]],   ← row 1 (contributes to output y)
      [M[2][0], M[2][1], M[2][2]] ]  ← row 2 (contributes to output z)
```

---

## 4. Rotation Matrices

All angles must be in **radians**. Convert degrees: `rad = deg * (Math.PI / 180)`.

### 4.1 Rotate Around the X-Axis (pitch — nod up/down)

```js
function rotXMat(a) {
  return [
    [1,            0,           0],
    [0,  Math.cos(a), -Math.sin(a)],
    [0,  Math.sin(a),  Math.cos(a)],
  ];
}
```

### 4.2 Rotate Around the Y-Axis (yaw — turn left/right)

```js
function rotYMat(a) {
  return [
    [ Math.cos(a), 0, Math.sin(a)],
    [           0, 1,           0],
    [-Math.sin(a), 0, Math.cos(a)],
  ];
}
```

### 4.3 Rotate Around the Z-Axis (roll — tilt sideways)

```js
function rotZMat(a) {
  return [
    [Math.cos(a), -Math.sin(a), 0],
    [Math.sin(a),  Math.cos(a), 0],
    [          0,            0, 1],
  ];
}
```

---

## 5. The Origin-Centering Rule (Critical — Most Models Skip This)

A rotation matrix always rotates around the **world origin (0, 0, 0)**.
If your object's center is at, say, `{x: 400, y: 300, z: 200}`, multiplying
its vertices directly by a rotation matrix will cause them to orbit the world
origin — a huge, broken arc off-screen.

**Always follow this three-step pattern for every local object rotation:**

```js
// center = the object's world-space center point {x, y, z}
// v      = the vertex to rotate {x, y, z}
// M      = a rotation matrix from Section 4

function rotateAroundCenter(v, center, M) {
  // Step 1: Translate so the object's center is at (0,0,0)
  const local = {
    x: v.x - center.x,
    y: v.y - center.y,
    z: v.z - center.z,
  };

  // Step 2: Apply the rotation (now around the correct center)
  const rotated = multMat(M, local);

  // Step 3: Translate back to world position
  return {
    x: rotated.x + center.x,
    y: rotated.y + center.y,
    z: rotated.z + center.z,
  };
}
```

---

## 6. Camera Transform

There is no "camera object" that moves through the world. Instead, the entire
world moves in the **opposite direction** of where you want the camera to go.
This is mathematically equivalent and simpler to implement.

### 6.1 Camera Translation

```js
// camera = {x, y, z}  — the camera's position in world space
// v      = a world-space vertex {x, y, z}

function applyCameraTranslation(v, camera) {
  return {
    x: v.x - camera.x,
    y: v.y - camera.y,
    z: v.z - camera.z,
  };
}
```

### 6.2 Camera Rotation (Optional)

If the camera has its own yaw/pitch, apply the **inverse** rotation to every
vertex. "Inverse" for a rotation matrix is its transpose (swap rows and
columns). For Euler angles, negate the angles:

```js
// If camera has yaw angle `camYaw`, apply rotation of `-camYaw` to all vertices.
const inverseCamRotation = rotYMat(-camera.yaw);
v = multMat(inverseCamRotation, v);
```

---

## 7. Perspective Projection

After camera transforms, convert 3D coordinates to 2D screen coordinates.

```js
// Parameters:
//   point         — {x, y, z} in camera space (after camera transforms)
//   fov           — field-of-view scalar (try 256–512; larger = narrower FOV)
//   viewerDist    — distance from viewer to projection plane (try 5–10)
//   screenW       — canvas width in pixels
//   screenH       — canvas height in pixels
//
// Returns {x, y, z} where x/y are 2D screen coordinates,
// or null if the vertex is behind the camera.

function perspectiveProject(point, fov, viewerDist, screenW, screenH) {
  const z = viewerDist + point.z;

  // Vertex is behind or at the camera — do not render.
  if (z <= 0) return null;

  const scale = fov / z;

  return {
    x: (point.x * scale) + (screenW / 2),   // center on screen horizontally
    y: (point.y * scale) + (screenH / 2),   // center on screen vertically
    z: point.z,                              // retain for depth sorting
  };
}
```

**How the formula works:**
- Larger `z` → smaller `scale` → vertex appears smaller (receding into distance).
- Adding `screenW / 2` and `screenH / 2` centers the world origin at the middle
  of the canvas.
- The raw `z` value is kept so faces can be sorted by depth (see Section 9).

---

## 8. Flat-Shading Lighting (Distance-Based)

Full raytracing is impractical for a CPU renderer. Use this approximation:

1. Define a light source at some world position `light = {x, y, z}`.
2. For each face (polygon), calculate its **center point** by averaging its
   vertices.
3. Compute the 3D Euclidean distance from the face center to the light.
4. Map that distance to a darkness value `[0.0, 1.0]`.
5. Draw the face in its base color, then overlay a black polygon at that alpha.

```js
// faceVertices — array of {x, y, z} for this face (3 or 4 vertices)
// light        — {x, y, z} light position
// maxDist      — distance at which the face is fully dark (e.g., 600)

function computeFaceDarkness(faceVertices, light, maxDist) {
  // Step 1: Find face center
  const n = faceVertices.length;
  const center = faceVertices.reduce(
    (acc, v) => ({ x: acc.x + v.x, y: acc.y + v.y, z: acc.z + v.z }),
    { x: 0, y: 0, z: 0 }
  );
  center.x /= n;
  center.y /= n;
  center.z /= n;

  // Step 2: Distance to light
  const dx = center.x - light.x;
  const dy = center.y - light.y;
  const dz = center.z - light.z;
  const dist = Math.sqrt(dx * dx + dy * dy + dz * dz);

  // Step 3: Clamp to [0.0, 1.0]  (0 = fully lit, 1 = fully dark)
  return Math.min(dist / maxDist, 1.0);
}

// Drawing (2D Canvas API example):
function drawShadedFace(ctx, screenPoints, baseColor, darkness) {
  // Draw base color
  ctx.fillStyle = baseColor;
  ctx.beginPath();
  ctx.moveTo(screenPoints[0].x, screenPoints[0].y);
  for (let i = 1; i < screenPoints.length; i++) {
    ctx.lineTo(screenPoints[i].x, screenPoints[i].y);
  }
  ctx.closePath();
  ctx.fill();

  // Overlay black at darkness alpha
  ctx.fillStyle = `rgba(0, 0, 0, ${darkness})`;
  ctx.beginPath();
  ctx.moveTo(screenPoints[0].x, screenPoints[0].y);
  for (let i = 1; i < screenPoints.length; i++) {
    ctx.lineTo(screenPoints[i].x, screenPoints[i].y);
  }
  ctx.closePath();
  ctx.fill();
}
```

---

## 9. Depth Sorting (Painter's Algorithm)

Without sorting, faces that are further from the camera will draw on top of
closer faces. Sort faces **after projection, before drawing**, using the
average Z of each face's projected vertices:

```js
// faces — array of face objects, each with a `projectedVertices` array {x, y, z}

faces.sort((a, b) => {
  const avgZa = a.projectedVertices.reduce((s, v) => s + v.z, 0) / a.projectedVertices.length;
  const avgZb = b.projectedVertices.reduce((s, v) => s + v.z, 0) / b.projectedVertices.length;
  return avgZb - avgZa;   // largest z (furthest) drawn first
});
```

---

## 10. Scanline Texturing (Advanced — CPU-Intensive)

Standard 2D APIs cannot warp or skew images onto arbitrary quadrilaterals.
To texture a face from scratch:

1. **Divide the projected face** into vertical strips (one pixel wide, or a few
   pixels for performance).
2. **For each strip**, calculate the top and bottom Y positions by linearly
   interpolating along the face edges.
3. **Map each strip's X position** to a corresponding column in the source
   texture image using the ratio: `texX = (stripIndex / totalStrips) * texWidth`.
4. **Draw a 1-pixel-wide slice** of the source texture scaled to fit the strip
   height using `ctx.drawImage(image, texX, 0, 1, texHeight, screenX, topY, 1, stripHeight)`.

> **Performance note:** Scanline texturing is CPU-heavy. For most CPU engines,
> use solid-color flat shading (Section 8) instead. Reserve texturing for
> engines targeting desktop hardware.

---

## 11. Common Failure Modes and Fixes

| Symptom | Cause | Fix |
|---|---|---|
| Object orbits off-screen when rotated | Rotating without centering origin | Apply Section 5 center–rotate–uncenter pattern |
| Moving camera causes world to jump | Forgot to negate camera offset | Subtract camera position from all vertices (Section 6.1) |
| Back faces drawn on top | Faces not depth-sorted | Sort by average Z descending before drawing (Section 9) |
| Vertices behind camera render | `z <= 0` not checked | Return `null` from `perspectiveProject` if `z <= 0` |
| Rotation wobbles/drifts over time | Angle accumulated as float indefinitely | Wrap angle: `angle = angle % (2 * Math.PI)` |
| Scene appears mirrored horizontally | Y-axis convention mismatch | Check Section 1; flip Y if your API uses +Y = up |
| Matrix result is wrong | Rows/columns transposed | Verify M[row][col] layout matches Section 3.1 exactly |

---

## 12. Minimal Working Example (JavaScript Canvas 2D)

A spinning wireframe cube to verify your pipeline is correct end-to-end:

```js
const canvas = document.getElementById('c');
const ctx    = canvas.getContext('2d');
const W = canvas.width  = 600;
const H = canvas.height = 600;

// 8 vertices of a unit cube centered at (0,0,0), side length 200
const baseVertices = [
  {x:-100, y:-100, z:-100}, {x: 100, y:-100, z:-100},
  {x: 100, y: 100, z:-100}, {x:-100, y: 100, z:-100},
  {x:-100, y:-100, z: 100}, {x: 100, y:-100, z: 100},
  {x: 100, y: 100, z: 100}, {x:-100, y: 100, z: 100},
];

// 12 edges as index pairs
const edges = [
  [0,1],[1,2],[2,3],[3,0], // front face
  [4,5],[5,6],[6,7],[7,4], // back face
  [0,4],[1,5],[2,6],[3,7], // connecting edges
];

const camera  = {x: 0, y: 0, z: -400};
const FOV     = 300;
const VIEWER  = 5;
let   angleY  = 0;

function multMat(M, v) {
  return {
    x: v.x*M[0][0] + v.y*M[0][1] + v.z*M[0][2],
    y: v.x*M[1][0] + v.y*M[1][1] + v.z*M[1][2],
    z: v.x*M[2][0] + v.y*M[2][1] + v.z*M[2][2],
  };
}

function rotYMat(a) {
  return [
    [ Math.cos(a), 0, Math.sin(a)],
    [           0, 1,           0],
    [-Math.sin(a), 0, Math.cos(a)],
  ];
}

function project(v) {
  const cam = {x: v.x - camera.x, y: v.y - camera.y, z: v.z - camera.z};
  const z   = VIEWER + cam.z;
  if (z <= 0) return null;
  const s = FOV / z;
  return {x: cam.x * s + W / 2, y: cam.y * s + H / 2};
}

function frame() {
  ctx.clearRect(0, 0, W, H);
  ctx.strokeStyle = '#00ff88';

  const M = rotYMat(angleY);

  // Rotate each vertex around cube center (which is already at 0,0,0)
  const projected = baseVertices.map(v => project(multMat(M, v)));

  for (const [a, b] of edges) {
    const pA = projected[a];
    const pB = projected[b];
    if (!pA || !pB) continue;
    ctx.beginPath();
    ctx.moveTo(pA.x, pA.y);
    ctx.lineTo(pB.x, pB.y);
    ctx.stroke();
  }

  angleY += 0.01;
  requestAnimationFrame(frame);
}

frame();
```

This minimal example intentionally omits texturing, lighting, and faces to keep
the pipeline verification simple. If the cube spins correctly in place with no
drift or mirroring, your core pipeline is correct.
