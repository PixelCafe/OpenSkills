---
name: unity-procedural-conveyor
description: Build or correct procedural 3D conveyor geometry in Unity, including open or closed paths, swept solid cross sections, low rounded rails, roller ends, surface normals, and belt motion. Use for geometry construction and visual verification; not for conveyor control logic alone.
---

# Procedural 3D conveyor

Build geometry from independent **path**, **cross section**, and **endpoint** specifications. Support open paths, closed loops, and custom curves. Do not interpret “conveyor” as necessarily a loop, a rectangle, or the particular shape in a prior example.

## Establish the shape contract

Inspect existing scene, scripts, render pipeline, and reference images before editing. Record these choices in the implementation:

- Path topology: open or closed; centerline control points/segments, orientation and travel direction.
- Cross section: usable belt width, deck height, bottom height, rail thickness and crown profile. Use Unity Y as height when feasible; document any coordinate transform.
- Open endpoints: independent start/end treatments (roller return, flat cap, discharge, or connection to another module). Closed paths have no endpoint assemblies.
- Appearance and motion: preserve existing material choices and animation unless the requested geometry needs adaptation. A painted belt arrow follows the surface; a transported object follows the transport route. They have different behavior at a roller.

Trace the complete route in the reference before choosing topology. Check the active scene and intended variant when several conveyors exist in the project. Make topology an explicit input shared by geometry and motion. For an open route, include the full terminal envelope when checking where the tail ends relative to the inlet.

Use supplied dimensions first. When unspecified, choose and state a practical parameterization; the [validated example](references/geometry-and-verification.md#reference-preset) is an optional preset, not a universal conveyor shape. Do not add a central board, UI, roller tray, or decorative objects merely because the prior example contained them.

## Generate the mesh

Read [geometry-and-verification.md](references/geometry-and-verification.md) for sweep equations, normal construction, end geometry, and acceptance checks.

1. Implement a centerline evaluated by distance, with position and tangent. Separate geometry resolution from animation speed. Use exact straight/arc evaluation or an arc-length lookup for splines.
2. Build a solid cross section and sweep the same section through straight and curved segments. Top, bottom, vertical facades, rail crowns, and required end closures are surfaces, not stacked flat outlines.
3. Apply the topology branch:
   - **Open:** terminate the sweep; generate only the requested end modules and their caps. No last-to-first surface triangles or phantom corner connecting endpoints.
   - **Closed:** join the final ring to the first with matching frame, cross section, normals and UV phase. Do not add inlet/outlet caps or a roller nose. A closed centerline does not imply an additional underside return path unless requested.
4. Split vertices at hard cross-section corners. Use smooth analytic or derivative-based normals on curved crowns and cylinders. Share or match normals across longitudinal tessellation boundaries. Ensure winding, culling, and depth work for a real 3D mesh.
5. Generate roller returns in the vertical travel plane, with the roller axis across the belt. Never substitute a horizontal semicircular end cap for a belt curling underneath.
6. Preserve surface-attached animation after geometry edits. Closed loops wrap distance continuously with constant marker/object scale across the seam. Open routes use explicit endpoint behavior. Painted arrows follow the actual belt skin around a support roller using the same surface evaluator as the skin. Recompute normals after deformation and share longitudinal vertices for smooth lighting.

When a receiving tray is requested, read the [terminal and belt-return guidance](references/geometry-and-verification.md#receiving-tray-and-discharge-belt-return). Match rail crown heights through the joint, carry the guardrails into the U-shaped rim, and close solid boundaries while preserving the designed opening.

For planar paths, require a bend radius greater than the furthest cross-section offset and check nonadjacent segments for collisions. For slopes or nonplanar routes, use a stable transported frame and reconcile closed-loop frame twist; do not reuse a planar side-vector formula blindly. Smooth abrupt path corners with a defined fillet or implement an explicit joint.

## Lighting and Unity lifecycle

Keep appearance work scoped. If an existing unlit shader prevents the requested normal-driven lighting, explain the conflict and make the smallest compatible lighting change, preserving base colors and textures. Do not silently redesign the material. Opaque depth-tested geometry should establish visibility; sorting orders are not a substitute for proper surface depth.

Surface-painted arrows use the belt's lighting model with a slightly lighter base color. Let them exist from the underside of a return and rely on depth and backface visibility. Remove hard visibility gates at the roller midpoint. If initial geometry still leaks, first fix the surface/occluder and use only a short continuous fade with an alpha-capable shader if needed. Check the roller-to-flat offset and normal continuity as well as brightness.

Use project tools where available; when operating Unity through MCP, follow the available Unity MCP skill for state, compilation and console checks. The geometry workflow must also work without MCP.

Procedural preview generation must own only its generated meshes, materials and child root. Do not delete arbitrary user children. Release owned resources safely. Never loop until childCount becomes zero while using deferred Destroy; hierarchy changes can also fail during parent activation/deactivation. Use bounded cleanup and a safe rebuild phase. Do not hide an editor hang behind repeated refresh calls.

## Verify before delivery

Use the user's camera specification. A tilted orthographic camera shows thickness while keeping parallel lines parallel; a single screenshot does not justify asserting a perspective camera. For the 70-degree reference, use the acceptance calculations in the reference document.

- Verify geometry dimensions from generated positions, not just from parameter values printed to a report.
- Verify normals, winding, seam continuity, and closures as relevant. A unit-normal check alone does not prove correct normal direction or a watertight solid.
- Inspect a full view and an endpoint/seam close-up. Check visible facades agree with view direction and are not equal-width borders everywhere.
- For a loop, inspect and animate across the seam. For an open route, inspect both endpoints and a marker entering/exiting the return surface.
- For an open terminal, inspect a 45-degree view and a true side view for rail height continuity, roller count/clearance and the full terminal/inlet placement. For lighting edits, record a short overhead view and an oblique nose view spanning underside-to-top travel, and inspect consecutive frames across the join.
- Compile, enter/exit Play Mode, and check the console after the final changes. Capture the actual final geometry.

Deliver the scene/prefab and source location, screenshot, selected dimensions/topology, and a short explanation of vertex rings and normals. State any unverified aspects. A Skill helps enforce this construction and verification process; it does not guarantee correctness without those checks.
