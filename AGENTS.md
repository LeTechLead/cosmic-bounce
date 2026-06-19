# Cosmic Bounce — AGENTS.md

## What this is

A single-file HTML5 Canvas game in the vein of Doodle Jump. Bounce a glowing orb up procedurally generated platforms, collect gems, build combos, and don't fall off the bottom of the screen.

**No dependencies, no build step, no framework.** Open `index.html` in a browser and it runs.

## File layout

```
cosmic-bounce/
  index.html   # The entire game — HTML + CSS + JS, zero build step
```

That's it. One file, ~495 lines, no build pipeline.

## Game design

- **Player**: Glowing orb, resets 3 mid-air jumps on every platform landing.
- **Controls**: `A` / `D` or `ArrowLeft` / `ArrowRight` steer horizontally; `Space` starts the game or uses a mid-air jump.
- **Scoring**: Purely altitude — score equals highest altitude (in metres, 5 px = 1 m), displayed as `XXm`. Gems and combos produce floating visual feedback (`+15`, `xN COMBO`) but **do not add points** to the score. The combo counter (🔥 in the top-right) increments on new platforms and gems, resets when re-landing on a previously visited platform.
- **Difficulty curve**: `diff()` returns 1–6 based on score. Higher difficulty = narrower platforms, slightly faster bounce.
- **Game over**: Player falls 100 px below the bottom of the camera view. High score persisted in `localStorage`.

## Code architecture

All code lives inside a single `<script>` block, organized into clearly marked sections using `// ─── Section ───` comments:

1. **Canvas** — setup and resize handler.
2. **Audio** — Web Audio API oscillator beeps (pentatonic-scale bounces, random-chime gem collects, descending sawtooth game-over).
3. **Particles** — emit / update / draw system with fade-out and drag.
4. **Background stars** — 250 stars with parallax. Three behaviors: normal twinkle, pulsar (bright rhythmic pulse), flasher (sharp sudden spike then fade).
5. **Floating text** — temporary pop-ups for gem collects (`+15`) and combo milestones (`xN COMBO`).
6. **Game State** — `state` enum (`start` / `playing` / `over`), score, combo, high score (localStorage).
7. **Difficulty & generation** — `diff()`, `addPlatform()`, `addGem()`, `initGame()`.
8. **Input** — `keydown` / `keyup` handlers for Space, A/D, ArrowLeft/ArrowRight.
9. **Update** — physics loop: gravity, movement, platform collision, gem pickup, combo logic (resets on re-landing same platform), procedural generation, cleanup, game-over check.
10. **Draw** — background gradient + nebulae + stars, platforms with glow, gems with pulsing animation, player with radial gradient + speed lines, particles, floating text, shake effect.
11. **Loop** — `requestAnimationFrame` loop calling `update()` then `draw()`.

## Conventions

- **No external libraries.** Everything is vanilla Canvas 2D + Web Audio API.
- **No classes.** All state is plain objects or arrays.
- **No IIFE wrappers** (except the RAF loop itself). All logic is at module scope inside the `<script>` tag.
- **No `const`/`let` for shared mutable state** at the top level — variables are declared at the appropriate scope with `let` for mutation.
- **Section headers** follow the `// ─── Name ───` pattern for easy navigation.
- **Color constants** are defined as arrays of arrays (`platColors`, `gemCols`) near the top of the script.

## Extending the game

### Adding features without breaking the single-file constraint

New sections go **before** the `// ─── Update ───` and `// ─── Draw ───` blocks so their data is available to the update and draw loops. Integration points:

- **New game objects**: add array + generator in the generation section, update logic in the Update section, rendering in the Draw section.
- **New audio cues**: add a `playX()` function in the Audio section, call from Update where appropriate.
- **New power-ups**: follow the gem pattern — spawn via `addPlatform`/`initGame`, check proximity in Update, render in Draw.
- **New platform types**: extend `addPlatform` with a `type` field; collision and rendering branches on it.

### Adding a build step

If the codebase grows and you want to introduce a build pipeline, the first step would be to split `index.html` into separate files and wire them up with a bundler (Vite recommended). Keep the `// ─── Section ───` comments as module boundary markers during extraction.

## Testing

Open `index.html` in any modern browser. There is no automated test suite.

## Deployment

The file is self-contained. Deploy by placing `index.html` on any static host:

- GitHub Pages (just push `index.html` to the repo root)
- Netlify (drag-and-drop the repo root)
- Any web server serving the file at `index.html`

No assets to copy, no build artifacts, no subdirectories.
