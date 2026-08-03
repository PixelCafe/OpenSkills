---
name: unity-ui-asset-exports
description: Create Unity-ready layered UI raster assets from a mockup or reference image. Use when extracting or recreating game buttons, icons, badges, panels, or other UI elements as aligned transparent PNG layers, especially when outputs need chroma-key removal, antialiased edges, a fixed target size, and a Unity compositing preview.
---

# Unity UI Asset Exports

Create reusable UI assets; do not modify gameplay code unless explicitly asked.

## Workflow

1. Inspect the reference image and identify the requested layers. Preserve the control's aspect ratio; use identical canvas dimensions and alignment for layers that must overlap.
2. For a low-resolution screenshot, recreate opaque UI layers at high resolution with `image_gen`; do not use screenshot-only segmentation as the final asset when it produces jagged edges.
3. Generate one source image per distinct layer on a uniform chroma-key background (`#00ff00` unless the subject is green). Specify the reference visual invariants: geometry, light direction, top/side treatment, shadow, and forbidden elements.
4. Copy selected generated sources into the project, remove the chroma key with `$CODEX_HOME/skills/.system/imagegen/scripts/remove_chroma_key.py`, then resize with Lanczos to the requested target size.
5. For foreground icon layers, resize and position their non-transparent artwork on a same-sized transparent canvas before exporting.
6. Do not create composited previews by default. Only create a preview when the user explicitly asks to verify the visual effect, preview the asset in context, or check stacked layers. Never overwrite an existing asset unless the user explicitly requested it.

## Defaults

- Target size: request one; otherwise use `256 x 256` for square buttons/icons. Use a tight aspect-preserving canvas for non-square controls: `512 x 160` is the default for a horizontal pill and `160 x 512` for a vertical pill.
- Render sources at least 2x the final dimensions, then downscale. Do not force a horizontal or vertical control into a square canvas merely to match another asset.
- Export PNG with RGBA and transparent corners.
- Use descriptive sibling names such as `<asset>_background_256.png`, `<asset>_background_512x160.png`, `<asset>_icon_256.png`, and `<asset>_preview_256.png`.
- For a project containing `techDocs/ui-asset-production.md`, follow that document's paths and acceptance criteria.

## Validation

Confirm before handoff:

- Every requested layer is RGBA, target-sized, and has transparent corners.
- No visible chroma-key fringe or screenshot aliasing remains.
- Aligned layers stack without offsets, clipping, or unexpected holes when a stacked preview is explicitly requested.
- When a preview is explicitly requested, it preserves the intended lighting hierarchy, especially attached sidewalls and drop shadows.

Report final paths, the target size, whether layers were generated or extracted, and the validation result.
