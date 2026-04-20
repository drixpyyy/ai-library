---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with high design quality. Use this skill when the user asks to build web components, pages, artifacts, posters, or applications (examples include experimental interfaces, creative portfolios, interactive visualizations, React/HTML/CSS layouts with unusual aesthetics, or any UI that demands a memorable, non‑generic presence). Generates creative, polished code that avoids generic AI aesthetics.
license: Complete terms in LICENSE.txt

This skill guides creation of frontend interfaces that feel authored, not assembled—interfaces that reject the homogeneous “AI aesthetic” in favor of unmistakable identity. The user provides frontend requirements; you deliver working, production‑grade code with obsessive attention to visual detail and creative risk.

## Design Thinking

Before writing any code, establish a bold conceptual anchor and commit to it completely:

- **Context & Purpose**: Who is this for? What emotional or intellectual reaction should it provoke? Is it a portfolio, a tool, a statement, an experiment?
- **Tonal Extreme**: Choose a clear, uncompromising direction—*glitch‑tech brutalist*, *fluid organic surrealism*, *harsh geometric abstraction*, *data‑moshed nostalgia*, *refined tactile luxury*, *algorithmic chaos*. Do not dilute. The chosen tone should feel like a deliberate world.
- **The Unforgettable Element**: Identify the single visual or interactive moment that will burn into memory—a shader‑driven background that reacts to sound, a navigation that fragments on hover, a typographic system that warps with scroll velocity, a custom cursor that leaves trails of light.
- **Technical Canvas**: Acknowledge constraints (frameworks, performance targets, accessibility) as creative parameters, not limitations.

The work must feel **intentional**. Whether maximalist and overwhelming or minimal and severe, every detail serves the chosen direction.

## Frontend Aesthetics & Implementation Principles

### Typography
- Avoid the default “safe” families (Inter, Roboto, Arial, system fonts). Choose typefaces with personality: sharp serifs, compressed grotesks, mono‑spaced peculiarities, variable fonts with axes you exploit (weight, slant, optical size).
- Pair a commanding display face with a restrained secondary font. Use extreme sizes, tight leading, or generous tracking to reinforce mood.

### Color & Atmosphere
- Commit to a dominant chromatic world with deliberate accents. Dark, brooding palettes with neon leaks; bleached monochromes with one saturated interruption; muddy, organic hues that feel excavated.
- Use CSS variables to maintain coherence across complex component trees.
- **Beyond flat color**: Introduce depth with layered gradients, noise overlays (`backdrop-filter` + SVG noise), or shader‑generated textures. The background should feel alive.

### Motion & Interaction
- Motion is narrative. Use staggered reveals on page load (`animation-delay` cascades) to create orchestration.
- Prioritize CSS for simple transitions; leverage Canvas/WebGL or libraries like **Motion** (React) or **GSAP** for complex sequences.
- **Scroll‑driven effects**: Parallax that disorients, clip‑path reveals, color interpolation tied to scroll progress.
- Hover states should surprise: perhaps a glitch displacement, a liquid distortion, or a dramatic scale shift that reveals hidden content.

### Spatial Composition & Layout
- Reject predictable grid conformity. Use asymmetry, overlapping elements, diagonal axis flows, and intentional breaks from container boundaries.
- Embrace either generous negative space (to amplify isolated elements) or controlled, dense information architecture that rewards exploration.
- Consider elements that bleed off‑canvas or interact with the viewport edges.

### Visual Details & Texture
- Add layers of atmosphere: subtle grain, scanlines, chromatic aberration on interaction, soft glows, harsh underlines, decorative borders that react to pointer position.
- Custom cursors that become part of the experience—large, playful, or context‑sensitive.
- Use `mix-blend-mode`, `filter`, and canvas compositing to create depth beyond standard DOM stacking.

### Anti‑Generic Commitment
- **Never** use purple‑gradient‑on‑white, default Bootstrap patterns, or centered hero text with a CTA arrow.
- **Never** rely on the same font twice across different generations. Vary between light and dark themes, cold and warm palettes, organic and mechanical motifs.
- If the output starts feeling “safe” or “like something I’ve seen on Dribbble a thousand times,” pivot harder.

### Technical Fidelity
- Code must be functional, responsive, and performant. Use semantic HTML, accessible ARIA where appropriate, and fallbacks for cutting‑edge features.
- For shader or WebGL work, provide complete, self‑contained examples that run in a browser without external dependencies (unless specified). Use Three.js or raw WebGL with clear, commented code.
- Match implementation complexity to the aesthetic vision: a brutalist interface might rely on harsh CSS grid and stark transitions; a fluid, dreamlike piece may demand fragment shaders and `requestAnimationFrame` loops.

## Execution Mindset

This skill exists to demonstrate what frontend can be when freed from conventional templates. You are not generating a website—you are authoring a small piece of digital culture. Make every pixel count.

Interpret requirements creatively. If the user asks for a “portfolio,” give them an immersive memory palace, not a grid of cards. If they ask for a “dashboard,” make it a glitch‑core monitoring station from an alternate timeline. Be unforgettable.
