# OpenSkills

Reusable skills for OpenAI Codex.

## Skills

### unity-ui-asset-exports

Creates Unity-ready layered UI raster assets from mockups or reference images. It helps Codex recreate or extract buttons, icons, badges, panels, and other UI elements as aligned transparent PNG layers.

### unity-procedural-conveyor

Builds procedural 3D conveyors in Unity with open, closed-loop, or custom curved paths. Covers solid cross sections, low rounded rails, roller returns, slope-aware frames, surface normals, and geometry verification. Reference dimensions are optional presets.

## Install

Copy the skill folder to your Codex skills directory:

```text
%USERPROFILE%\.codex\skills\unity-ui-asset-exports
```

Then invoke it in Codex:

```text
Use $unity-ui-asset-exports to create layered Unity UI sprites from this reference image.
```

For procedural conveyors, copy `skills/unity-procedural-conveyor` to `%USERPROFILE%\.codex\skills\unity-procedural-conveyor`, then invoke it:

```text
Use $unity-procedural-conveyor to create a closed-loop 3D conveyor with a ramp in Unity.
```

This repository contains the Codex skill definition only. Scripts provided by Codex or other external dependencies are not bundled.
