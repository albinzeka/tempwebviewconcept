# Concept Morph Viewer

Browser morph / proportion concept viewer (no Flutter required).

## Live

**https://albinzeka.github.io/tempwebviewconcept/**

Includes Bob hair + silhouette outline (`best` / `ssaa2`) as of 2026-08-16.

If that 404s, enable Pages once: **Settings -> Pages -> Branch `main` -> folder `/ (root)` -> Save**.

## Files

| Path | Role |
|---|---|
| `index.html` | Viewer + morph UI + outline |
| `models/height_overlay_clean.glb` | Character |
| `models/bob.stripped.glb` | Bob hair |
| `textures/` | Skin / face / eye maps |
| `vendor/` | Three.js |
| `SLIDER_RECIPES.md` | Bone / shapekey recipes |

UI labels show only `(Bn)` / `(Sk)`. What's new is in the viewer panel.
