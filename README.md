# Concept Morph Viewer

Browser morph / proportion concept viewer (no Flutter required).

## Live

**https://albinzeka.github.io/tempwebviewconcept/**

Includes as of 2026-08-17:

- Bob hair, silhouette outline (`best` / `ssaa2`)
- Manual texture load (session-only overlays)
- Eye / hair color (preset + hue)
- Hair + skirt spring physics (character spin, locked scalp roots, sway cones)

If that 404s, enable Pages once: **Settings -> Pages -> Branch `main` -> folder `/ (root)` -> Save**.

## Files

| Path | Role |
|---|---|
| `index.html` | Viewer + morph UI + physics |
| `models/height_overlay_clean.glb` | Character |
| `models/bob.stripped.glb` | Bob hair |
| `textures/` | Skin / face / eye maps |
| `vendor/` | Three.js |
| `SLIDER_RECIPES.md` | Bone / shapekey recipes |

UI labels show only `(Bn)` / `(Sk)`. What's new is in the viewer panel.
