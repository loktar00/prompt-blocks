# Character model block

Builds humanoid characters from code as one skinned mesh with layered gear. Paste it into any scene prompt that has characters; the walk block builds on it.

```text
BUILD EVERY HUMANOID CHARACTER FROM CODE, AS ONE SKINNED MESH WITH LAYERED GEAR, so it reads as a person with a clear silhouette at game distance and holds up close. No imported models, no box-and-sphere dolls. Build it exactly as described.

WHAT IT MUST DO
- One character = one skeleton, one geometry (all parts merged), skinned on the CPU or GPU, one shadow-casting mesh.
- The body is smooth lofted forms (torso, arms, legs, boots, head), never stacked primitives. Gear is built on top as separate shells that follow the body.
- Each character is identifiable by silhouette alone: its own headgear, shoulder shape, cloak or skirt line, weapon and carried object.
- Surfaces look like their material (cloth, leather, mail, plate, fur, skin, wood) with no image files.

1. SKELETON, hero scale 1.8 m tall, Y up, facing +Z, standing rest pose. Bone rest positions (y): root 0; hips 0.98; spine 1.12; chest 1.28; neck 1.5; head 1.585. Shoulders at x = +-0.2, y 1.435; elbows y 1.14; hands y 0.885. Hip joints at x = +-0.095, y 0.93; knees y 0.5; ankles y 0.085. Leg length hip to ankle is 0.845. Add extra bones for anything that must swing (cloak as 3 chained segments, lantern, tail, pack). Hang weapons on a hand bone. Vary characters by scale and a few numbers (shoulder half-width, hip half-width, height), not by a new rig.

2. ONE LOFT FUNCTION builds almost everything. Input: rings from bottom to top. Each ring has height y, half-widths rx (side), rz (front), rb (back, so front and back differ), centre offset x and z, and a squareness exponent (2 = ellipse, 2.5 to 3 = rounded box). Smooth between rings with Catmull-Rom (3 sub-steps per ring), 10 to 18 segments round. Options: arc (open shell, for plates and cloth panels), thickness (turns a surface into a thin shell), caps, and a small noise wobble on the radius for worn cloth. A lathe is a loft with only rx. A free-form surface builder (grid of u, v to a point) covers cloaks, skirts, shields and hoods.

3. BODY RINGS (y: rx / rz / rb). Torso: 0.90: .168/.105/.115; 1.00: .17/.11/.12; 1.10: .153/.105/.10; 1.27: .182/.125/.11; 1.37: .198/.122/.112; 1.44: .19/.105/.105; 1.49: .11/.075/.085 (neck base). Arm radius .066 at the shoulder, .047 at the elbow, .034 at the wrist. Leg radius .086 to .095 at the hip, .056 at the knee, .038 at the ankle. Boot: leg shell grown 9 mm, a folded cuff, and a foot lofted along Z with a flat sole and a raised heel, plus a thin dark sole shell. Hand: a fist loft, four small finger spheres, one thumb sphere. Head: a loft about 0.26 m tall with separate nose, brow and cheek planes, dark eye sockets as small flattened spheres, optional beard loft and hair cap. Keep the face restrained and shadowed.

4. SKIN WEIGHTS. Every vertex gets two bones and a blend weight set per part by a function of the vertex's rest height: across a joint, weight = 1 - smoothstep(jointY - r, jointY + r, y), r about 0.05 at elbows and knees, 0.035 at the neck. Skirts and cloaks blend hips into thighs by height (about 0.25 m below the waist they are fully on the thighs) so legs do not poke through. Rotate the rest normals with the same blend.

5. GEAR LAYERS, each a loft or surface grown 4 to 30 mm off the body and bound to the same bones: tunic (torso loft grown 27 mm, billowed 14 mm at the waist), skirt panels with sine fold noise at the hem, belt (grown 36 mm, buckle, strap end), pouches, a baldric as a flattened tube, shoulder plates as 2 to 3 overlapping curved lames split across chest and arm bones, elbow cops, vambraces, greaves and knee plates as arc-limited shells, helmets and hoods as lofts with the face opening cut by arc, a cloak as a surface with fold frequencies 9 and 19, a ragged hem and a clasp. Give each character 5 to 8 layers. Two-colour cloth (split down the middle) and one trim colour are the cheapest way to make a silhouette readable.

6. WEAPONS AND CARRIED THINGS, built in the hand's frame: sword blade with a diamond section (half-width 4.5 mm by 26 mm, tapering to a point), leather grip, guard and pommel sphere; shield on the forearm bone with a curved face, painted halves as two thin surfaces just proud of the board, rivets round the rim, straps on the back; bow, staff or lantern on their own bone so they can swing. Record weapon tip and base points in bone space so effects can follow them.

7. ONE MATERIAL SHADER for the whole mesh. Per vertex store a material id (0 skin, 1 cloth, 2 quilt, 3 mail, 4 leather, 5 plate, 6 fur, 7 wood), roughness, metalness and a wear amount. Pick the pattern in the fragment shader by id, from procedural noise: cloth weave from two sines at about 1500 per unit plus fold noise; mail from rows of abs(sin) * abs(sin) rings at about 350 per unit; leather creases; plate with scratches, dents and a rust mask; fur as long streaked noise; wood grain stretched along the length. Use the pattern for albedo variation (0.7 to 1.3 times) and as a height for bump mapping through screen-space derivatives. Darken with mud toward the feet. Tint every part from a palette of 5 to 8 muted colours with one accent.

BROWSER SAFETY: Windows Chrome runs WebGL through D3D11. Do all procedural texture work in the fragment shader. No texture reads in vertex shaders. No dynamic indexing of uniform arrays in vertex shaders. Vertex attributes are floats only (store bone indices as floats, or skin on the CPU). Constant loop bounds.

```
