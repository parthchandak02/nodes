# nodes — Creative Brief

## Concept
A spring-mass physics network where each node is a random person (from RandomUser API). Nodes settle into position via spring forces, forming clusters by country. Clean monochrome lines on dark background. Tap a node to reveal the person's full profile.

**Design Read:** Canvas generative art for personal viewing, physics simulation (spring-mass network), Void monochrome aesthetic, data-driven by random user profiles.

**Narrative Pattern:** Annotation — each node shows the person's first name + age inline at rest, always visible.

**Tier 2 Patterns (pick 2):** Quiet Zone (tap for full user details) + Hero Number (total user count displayed)

## Three Dials
- DESIGN_VARIANCE: 7 — semi-organic spring positions, slight cluster offsets, not perfect grid
- MOTION_INTENSITY: 5 — gentle settling, spring oscillations, slow damping
- VISUAL_DENSITY: 5 — moderate, 5-12 nodes with connecting lines

## Data Source
- API endpoint: `https://randomuser.me/api/?results=10`
- Response format: JSON with `results[]` array. Each result: `name.first`, `name.last`, `dob.age`, `location.country`, `gender`, `email`, `picture.thumbnail`
- Fallback: 5 hardcoded realistic user objects if API fails

## Visual Style
- Preset: Void — Monochrome (from visual-style-presets.md)
- Palette (JS const):
  ```js
  const PALETTE = {
    bg: '#0d0d0d',
    layer: '#1a1a1a',
    accent1: '#e0e0e0',
    accent2: '#9e9e9e',
    accent3: '#ffffff',
    glow: 'rgba(255,255,255,0.08)',
    text: 'rgba(255,255,255,0.6)',
  };
  ```
- Coherence tokens:
  - strokeWeightScale: {hair:0.5, fine:1, medium:2, heavy:4}
  - alphaHierarchy: {bg:0.08, field:0.25, active:0.7, label:0.45}
  - shapeLanguage: geometric (circles for nodes, straight lines for springs)
  - glowBudget: 1 glow color (white), blur ≤ 20px, ≤ 10 entities/frame

## Encoding Contract
| Data Field | Visual Channel | Range | Inverse Function | "Bigger means..." | Legend? |
|---|---|---|---|---|---|
| user.dob.age | node radius | 6-24px | invertScale(pixel, 6,24, 18,70) | older person | no |
| user.location.country | node initial position cluster | grouped by country | hitTest(x,y) → nodeId | same country = same cluster | yes (country labels at cluster centers) |
| user.gender | node shape | circle (male), square (female) | hitTest on node → check shape | gender | yes (legend shows shape meaning) |
| user.name.first + dob.age | always-visible label | text at 10-11px near node | — | who this person is | no |

## Interaction
- Tap: reveal Quiet Zone detail panel with full user profile (name, age, country, email, gender)
- Auto-animation: spring physics settles into equilibrium, gentle oscillation at low amplitude
- Long-press (1s): fetch 5 new users from API and replace
- After 3s idle: Quiet Zone fades back to clean state

## Communication Requirements
**Tier 1 (mandatory):**
1. Persistent Data Stamp — bottom left: "RandomUser API · N profiles · live/fallback label"
2. Data Process Honesty — if fallback data used, stamp says "fallback", never "live"
3. One-to-One Sensory Mapping — age → radius (wider = older), gender → shape (circle/square), country → cluster proximity

**Tier 2 (pick 2 — state which):**
4. Quiet Zone Detail Panel — DOM overlay at tap point shows: name, age, country, email, thumbnail. Fades after 3s idle.
5. Hero Number — total user count displayed as large number (48px) in top area with "people" label

**Coherence:**
6. Generative-art axes: strokeWeightScale {hair:0.5, fine:1, medium:2, heavy:4}, alphaHierarchy {bg:0.08, field:0.25, active:0.7, label:0.45}, shapeLanguage geometric, glowBudget 1 color white blur ≤20px ≤10 entities
7. Motion tokens: --motion-fast:160ms, --motion-base:240ms, --motion-slow:360ms, --ease-out-expo
8. AI Slop Score < 30 — no Inter/Roboto/Open Sans, no purple gradient, layered bg (layer color for spring lines), at least spring oscillation animation

## Canvas Code Requirements (MANDATORY)
1. Single HTML file, no external dependencies, no CDN libraries
2. Config object `C` at top with all tunable values
3. DPR-aware canvas setup: `Math.min(window.devicePixelRatio || 1, 2)`, `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)`
4. Canvas CSS: `position:fixed;top:0;left:0;width:100%;height:100%;display:block`
5. Touch guard pattern — `let tapping=false; ontouchstart` gate
6. First frame never blank — pre-run physics warmup (20 iterations) before first render
7. Realistic fallback data if API fails — 5 hardcoded realistic user objects
8. Short description text (`#desc` div, bottom center, muted opacity, 11px system font): "A spring-mass network of random people. Tap a node for details."
9. Full-viewport, mobile-responsive
10. Works offline after first load (no CDN deps)
11. `.nojekyll` file at project root
12. Chrome positions: stamp at bottom:20px left:16px, legend at bottom:44px center, desc at bottom:68px center — NO OVERLAP

## Interaction Affordances
- On first visit, show a brief visible hint text in the Quiet Zone area: "Tap a node to learn about them" for 3 seconds, then fade
- After 3 seconds of user inactivity (no tap), any visible detail panel fades back to clean art state
- Nodes have subtle pulsing glow on their accent ring (1-2px white ring with shadowBlur) to indicate they are interactive
- On tap: the node briefly flashes white (100ms), then the Quiet Zone appears

## Spring Physics Parameters
- Rest length: configurable (default 120px between connected nodes)
- Spring constant k: 0.008
- Damping factor: 0.85
- Repulsion force between all pairs: 500 / distance^2
- Connection topology: nearest-neighbor within radius (connect nodes within 200px distance in initialized positions)
- Velocity cap: 5px/frame max to prevent explosion
- dt: 0.05 for Euler integration stability
- Warmup iterations: 40 (run before first render so layout is partially settled)

## Quiet Zone Panel
- Position: near tapped node (choose quadrant to stay in viewport)
- Background: `#1a1a1a` with 1px `#e0e0e0` border, 8px border-radius
- Content: user's thumbnail image (32x32 from `picture.thumbnail`), name, age, country, email
- Font: 11px system-ui `.text-secondary`, name at 14px bold
- Fade: opacity 1 → 0 over 300ms after 3s idle timeout
