# Geometry and verification

## Path-independent sweep

For a planar horizontal path, use arc length s, position C(s) in XZ, unit tangent T(s), vertical U=(0,1,0), and transverse S=normalize(U cross T). The sign of S is arbitrary if used consistently. A cross-section vertex (q,h) becomes:

    V(s,q,h) = C(s) + q*S(s) + h*U

A profile normal (nq,nh) becomes normalize(nq*S + nh*U). Top normals are U; bottom normals are -U; planar vertical walls use +/-S. Duplicate vertices between top and wall to keep a hard edge. For a circular rail crown centered at (qc,hc):

    q = qc + r*cos(a)
    h = hc + r*sin(a)
    N = cos(a)*S + sin(a)*U,  a in [0,pi]

Use several crown subdivisions, with matching analytic normals at adjoining strips. Increasing longitudinal resolution should improve curved paths without introducing lighting bands. Include exact segment junctions in the sampled path. Uniform global samples alone can miss the line-to-arc junction and slightly cut corners.

For adjacent rings, connect the same profile edge using two triangles. Check triangle orientation against its intended outward normal. Avoid zero-area triangles at cylinder centers by using a triangle fan instead of collapsing an entire grid row to one point. When using separated face patches, geometric coincident edges can be watertight even though vertex indices differ for normals or UVs; check positions with a tolerance when evaluating closure.

For nonplanar paths use a parallel-transport frame. A cross-section twist is a design parameter, not an accidental Frenet-frame flip at an inflection. A closed nonplanar sweep may accumulate frame twist; distribute correction over arc length or honor an explicitly authored roll field.

## Open versus closed

Open paths have distinct first and last rings. End modules must meet the actual ring positions and profiles, including rail edges. Close exposed bottoms and end faces; avoid hidden gaps between a rail, body and roller cheek. Specify an underside return route only if needed for visible animation or geometry.

Preserve intentional openings when closing the solid boundaries. A requested open tray has a mouth and recessed floor, with wall thickness around that opening. A full end plate across the opening changes its structure.

Closed paths use one geometric seam. Either connect last unique ring to first, or duplicate the first ring at s=L for UVs; do not create an extra degenerate segment. At the seam, positions, tangents and cross-section frames agree. Match normals even when vertices are duplicated for UVs. If a periodic texture repeats N times around length L, its phase must agree modulo N at the seam. Motion uses s=Repeat(s0+speed*time,L).

Keep straight, circular, rounded-rectangle and spline loops as alternative path providers, not hardcoded generator variants. Check both local radius and global clearance; a wide sweep can self-intersect even when its centerline does not.

On a closed sampled path with a duplicated final ring, compute the first and last tangents using periodic neighbors. Match the transported frame after any residual twist correction. Keep endpoint caps, endpoint fading and return-roller assemblies conditional on open topology. Assign slope profiles by segment or arc-length interval so a newly added outer return stays at its intended elevation without abrupt height thresholds.

## Roller return at an open start

Let C0 be the start centerline point at ground height, T0 point into the top route, S0 be transverse, H be deck height, and R=H/2. A semicylindrical return from top to bottom is:

    P(a,q) = C0 - R*sin(a)*T0 + q*S0 + (R+R*cos(a))*U
    N(a)   = -sin(a)*T0 + cos(a)*U
    a in [0,pi]

Thus the top is at H, bottom at zero, and the outermost point is R behind C0. The axis is horizontal and perpendicular to travel. This is a vertical-plane half-circle; in plan view it is not a round cap across the full belt width.

The rail ends wrap along the same return region and must match the swept crown's height, width and normals. Explicitly specify how the small crown transitions around the roller, rather than increasing the whole belt's roll radius. If crown height varies across q, the end-surface normal needs both derivatives:

    N = normalize(cross(dP/da, dP/dq))

Orient the cross product outward. Reusing only the cylinder radial normal causes a visible seam where the crown meets the end surface. Include cheeks to close side faces.

Axle caps are discs centered on the roller axis at height R on both sides. Their planes are perpendicular to S0. Add a short cylinder rim for protrusion thickness. Keep cap normals flat on the face and radial on the edge, with duplicated vertices at that hard corner.

Painted markers use this same surface function. Marker longitudinal extent may be sampled into multiple vertices so an arrow bends around a small roller instead of rotating as one rigid card. The lower face is hidden by actual depth/culling; do not draw underside arrows above opaque geometry using a forced sorting order.

Use a matching lit material for belt and painted markers, preserving the belt's material settings and making only a slight base-color adjustment. After assigning deformed vertices, call Mesh.RecalculateNormals or supply equivalent smoothly varying surface normals. Shared vertices along each arrow stroke allow recalculated normals to blend across its longitudinal subdivisions. Keep the marker offset equal at the curved/flat join. Test winding with the lit material's backface culling enabled. Start the marker on the underside and let geometry occlude it throughout the rotation.

## Receiving tray and discharge belt return

This is an optional end-module pattern for an open conveyor. Fit the module to the selected route and reference. Define cell length separately from belt width when the project uses different units.

- The tray extends from the final belt segment. Its left, right and far rim form a U whose open side faces the incoming belt. Preserve crown height at the belt-to-rim joint and around both far corners. Widen the tray in plan without lowering the crown.
- Build the recessed floor, outer and inner walls, bottom, rounded crown, and roller end discs as actual surfaces. Choose a meaningful floor thickness and close their exposed boundaries.
- A useful reference layout has outer width 1.4 for belt width 1, length 1.7 cells, floor at 0.75 for deck height 1, and six transverse white rollers. Roller radius is approximately width/9, roller tops are just below deck height, and their lengths nearly fill the inner width. These values are configurable.
- A later correction requires the tray rim top to equal the main guardrail top, including its low rounded crown. With the reference cross section this is 1.067. An earlier deck-height rim produced a visible step. Verify the final choice in a side view.
- Fit roller centers with equal spacing and allow for their radii and the far inner wall. Count all physical rollers, including the support underneath a belt return.
- Keep the first roller and the curling belt skin as distinct geometry. Bridge the flat deck tangentially to the wrap, then curl the belt downward around the support. Add skin edge thickness as visible. The arrow follows the skin instead of being painted on the white roller.

For a roller center height c and deck height H, choose belt outer radius Rb=H-c. Require Rb greater than the support roller radius. For deck height 1, support radius 1/9 and roller top 0.98, c=0.868889 and Rb=0.131111. If the first roller center is at distance d0 beyond the deck end, use a flat bridge at height H for 0<=d<=d0. Then evaluate the outer skin by arc length s:

    a = (s-d0)/Rb
    distance = d0 + Rb*sin(a)
    height = c + Rb*cos(a)
    normal = sin(a)*T + cos(a)*U
    a in [0,pi]

Skin and arrow evaluators share this definition. Offset the arrow along this normal by a small consistent distance, allowing its mesh to bend over the wrap. Occlude the returning underside with the tray/belt structure. Using the white roller radius for the arrow while the deck ends at height H introduces a geometric gap and the wrong apparent attachment.

Check the terminal's full world-space bounds against the inlet and neighboring sections. When matching a reference where the tail stops short of the inlet, place the module first and derive the last belt endpoint from that module length. Extending a terminal from an old endpoint can overshoot. Use an overview for this relationship and close-ups for construction.

## Reference preset

This preset records one accepted stylized conveyor. Use only when requested or as a stated starting point. It does not constrain topology: the same cross section can run along open, closed or custom paths.

| Parameter | Value in belt-width units |
|---|---:|
| Usable top span | -0.5 to +0.5 |
| Ground / deck | y=0 / y=1 |
| Right rail extent | q=0.507 to 0.676 |
| Left rail extent | mirrored |
| Rail center | abs(q)=0.5915 |
| Crown radius | 0.0845 (approximately 0.085) |
| Crown circle center height | 0.9825 |
| Highest rail point | 1.067 |
| Start return radius / center height | 0.5 / 0.5 |
| Axle cap radius | 0.25 |

Resolve the small 0.007 gap between the specified top edge and rail inner face deliberately: use a narrow support/shoulder or an intentional seam appropriate to the model. Do not report a 1.014-wide deck as exactly 1.0. The side facade continues to ground; the visible rail above the deck is only 0.067, not a tall wall. The reference open path has three rounded turns and distinct start/end assemblies; a closed version must omit both assemblies and complete its path.

## Camera and visible-thickness check

For a belt in XZ, place an orthographic camera looking from negative Z toward the center, elevated 70 degrees above horizontal (20 degrees from the downward vertical). Keep camera roll zero. Point at the conveyor center; frame it using orthographic size rather than switching to perspective.

For unit height H=1, projected vertical facade thickness is H*cos(70)=0.342 relative to a world unit across screen horizontally. A unit ground-plane strip running toward the camera projects to sin(70)=0.940, so their ratio is cot(70)=0.364. Both match the requested approximately one-third appearance, but use the correct denominator when measuring screenshots.

Expected visibility: front/bottom outer facade and rear/top inner facade visible; the straight left/right facades almost edge-on. Curves transition smoothly. Equal-width dark borders around every segment indicate flat outline geometry or an incorrect camera, not a successful thickness test.

Orthographic projection may be used with any camera tilt. Do not conflate an inclined view with perspective. An axle cap viewed obliquely will project to an ellipse even though its geometry is a true circle.

## Acceptance cases

Exercise the cases relevant to the requested edit, and use these as future regression scenarios:

- Open U-shaped route: no hidden start-to-end connector; radius/height/axis of the return correct; markers roll up from underneath.
- Rounded rectangular loop: no endpoint hardware; position/tangent/normal/UV continuity at seam; marker crosses seam without a jump.
- Circular loop: constant width, continuous curvature, no wedge at seam.
- Custom spline: acceptable chord error, smooth transported frame, no inside-edge inversion or nonadjacent collisions.
- Lighting: unit normals plus outward orientation and hard/smooth boundaries checked; use a temporary neutral directional light if necessary. Keep the final requested material appearance.
- Return lighting: inspect short overhead and oblique recordings. The underside should emerge from occlusion in shadow, brighten continuously as normals turn toward the light, and meet the flat deck without a visibility, offset or brightness jump.
- Receiving tray: verify the open side faces the belt, three rim sides connect at equal crown height, six rollers fit with clearance when specified, and the purple belt skin and arrow turn down around the first support together. Compare a 45-degree view, a true side view and an overview showing the inlet/tail relationship.
- Lifecycle: edit preview, rebuild, enable/disable, domain reload, enter/exit play without duplicate geometry, deletion of user content or orphaned native meshes.

Measure generated mesh positions for dimensions. A report that merely echoes the expected constants is documentation, not validation. Visual acceptance supplements, rather than replaces, those geometry checks.
