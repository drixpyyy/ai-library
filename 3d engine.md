---
name: 3d-graphics-from-scratch
description: "Use this skill when building, reasoning about, or debugging 3D graphics engines from scratch using pure math and 2D drawing contexts (e.g., HTML5 Canvas 2D, Pygame) WITHOUT hardware acceleration (WebGL, OpenGL, Three.js). It acts as a golden reference for First Principles 3D: vertex projection, 3x3 matrix multiplication, Euler rotations, software-based texturing (scanline), camera logic, and flat-shading lighting calculations."
compatibility: "Any language/framework supporting basic 2D line/polygon drawing (JavaScript Canvas, Python Pygame, Lua LÖVE, C++ SDL)."
license: Proprietary. LICENSE.txt has complete terms
---
Building a 3D Graphics Engine from Scratch (CPU/Software Rendering)
Why this skill exists

When asked to create or debug 3D graphics, AI models natively default to relying on hardware-accelerated libraries (WebGL, Three.js, OpenGL). When forced to render 3D on a 2D canvas using pure code, models often hallucinate the matrix math, fail to handle camera perspectives correctly, or mess up the origin point during rotations.

This document provides the exact mathematical foundation and step-by-step rendering pipeline required to project 3D objects onto a 2D screen, including camera movement, lighting, and software-based texturing.

1. The Rendering Pipeline

To render a 3D frame from scratch, operations must occur in this exact order for every vertex:

Camera Translation: Move the vertex in the opposite direction of the camera.

Camera Rotation: Rotate the vertex around the camera's angles (inverse).

Object Translation/Rotation: Centralize the object to (0,0,0), apply its local rotations, then move it to its world position.

Perspective Projection: Convert the (x, y, z) coordinates into 2D screen coordinates (x, y).

Draw: Connect the projected 2D coordinates using lines (wireframe) or filled polygons (solid).

2. Matrix Multiplication (The Core Engine)

3D transformations are achieved by treating [x, y, z] as a 1D matrix and multiplying it against a 3x3 transformation matrix. (Note: standard engines use 4x4 matrices with a homogeneous coordinate w to handle translations easily, but for a simple CPU engine, 3x3 with manual offsets is faster).


function multMat(m, v) {
    return {
        x: v.x * m[0][0] + v.y * m[0][1] + v.z * m[0][2],
        y: v.x * m[1][0] + v.y * m[1][1] + v.z * m[1][2],
        z: v.x * m[2][0] + v.y * m[2][1] + v.z * m[2][2]
    };
}
3. Rotation Matrices

To rotate an object, you multiply its vertices by these specific Euler angle matrices. The angles must be in radians.

Rotate X Matrix:

const rotXMat = (angle) => [
    [1, 0, 0],[0, Math.cos(angle), -Math.sin(angle)],[0, Math.sin(angle), Math.cos(angle)]
];

Rotate Y Matrix:


const rotYMat = (angle) => [
    [Math.cos(angle), 0, Math.sin(angle)],[0, 1, 0],
    [-Math.sin(angle), 0, Math.cos(angle)]
];

Important - Centralizing the Origin:
If you multiply a vertex by a rotation matrix, it rotates around (0,0,0). If your object is at x: 400, it will orbit massively off-screen.
Fix: Subtract the object's center from the vertex, apply the rotation, then add the center back.


// Centralize -> Rotate -> Move Back
let centered = { x: v.x - center.x, y: v.y - center.y, z: v.z - center.z };
let rotated = multMat(rotYMat(angle), centered);
let movedBack = { x: rotated.x + center.x, y: rotated.y + center.y, z: rotated.z + center.z };
4. Camera & Perspective Projection

You do not move the camera; you move the entire world in the opposite direction.


let translated = {
    x: v.x - camera.x,
    y: v.y - camera.y,
    z: v.z - camera.z
};

Perspective Formula:
Objects further away (higher z) should appear smaller. We achieve this by dividing the x and y coordinates by z.


function perspectiveProject(point, fov, viewerDistance) {
    const z = viewerDistance + point.z;
    
    // Do not render vertices behind the camera
    if (z <= 1) return null; 
    
    const scale = fov / z;
    
    return {
        x: (point.x * scale) + (canvas.width / 2),
        y: (point.y * scale) + (canvas.height / 2),
        z: point.z // Kept for depth sorting later
    };
}
5. Software Rendering Challenges
Texturing (The Scanline Technique)

Standard 2D APIs cannot natively skew or warp images onto 3D quads. To map a texture from scratch:

Divide the 3D face into vertical or horizontal slices (scanlines).

Calculate the distance and angle between the top and bottom of the projected face.

Draw 1-pixel wide strips sliced from the source image, stepping across the face.
(Note: This is highly CPU intensive. For basic engines, stick to solid colors).

Lighting (Flat Shading)

True raytracing is too heavy for standard CPU rendering. Use distance-based flat shading instead.

Define a Light Source coordinate (lx, ly, lz).

Find the center point of the face you are drawing (average the x, y, z of its 3 or 4 vertices).

Calculate the 3D distance between the face's center and the light source:


let dx = faceCenter.x - light.x;
let dy = faceCenter.y - light.y;
let dz = faceCenter.z - light.z;
let dist = Math.sqrt(dx*dx + dy*dy + dz*dz);

Map this distance to a brightness value (e.g., 0.0 to 1.0).

Draw the face with its base color, then draw a black polygon over it with its globalAlpha set to the darkness level. The further the distance, the higher the black opacity.
