Create one complete, self-contained HTML file for WALK TEST, a small real-time three.js scene whose only purpose is to prove the character models and their procedural walk, run and fight animation. Keep the scene tiny so all the effort goes into the characters.

THE SCENE, deliberately small:
- A square courtyard of worn flagstones about 30 units across, with a subtle tiled texture drawn in code, floating on a dark background (#16171b).
- A few simple props: a low well in the middle, two benches, a gentle ramp up to a raised platform about 0.4 units high along one side, and a single 0.2-unit kerb step. Nothing else.
- One warm sun about 40 degrees up casting real shadows from a shadow map, plus a dim cool ambient. Each character casts a crisp shadow, so planted feet visibly meet their shadows.
- Camera: a three-quarter view from above that frames the whole courtyard. Mouse drag orbits, the wheel zooms, and it never goes below the ground. A "side view" button cuts to a low camera level with the selected character, tracking it from the side.

THE CHARACTERS: three humanoids with clearly different builds and gear, each built and animated exactly as the two blocks below describe:
1. A tall knight in plate and a tabard, sword at the hip and a shield on the back.
2. A slender traveller in a hooded cloak, carrying a lantern on a pole.
3. A short, stocky labourer in a leather apron, carrying a hammer.

HOW THEY MOVE, by default, with no input:
- The knight walks a slow figure-of-eight round the well at walking speed, steadily, forever.
- The traveller follows a loop that climbs the ramp, crosses the platform, steps down the kerb and returns. It cycles walk, then stop and stand for 2 s, then walk again.
- The labourer cycles stand, walk, run, sudden stop, then a short fight stance with a hammer swing, then walk off again, on a loop of about 20 s.
The user can take over: click a character to select it (a soft ring under its feet), then W A S D or the arrow keys move it, and Shift runs. After 5 seconds without input it eases back onto its path.

A SMALL PANEL, top-left, plain and tidy:
- a speed slider for the selected character, from standing through walk to run;
- slow motion 1x / 0.25x;
- toggles for "foot-pinning IK", "knee spring", "heel lift", "show foot contacts" (a small dot on each planted foot that turns red if the foot slides) and "show skeleton".
Each toggle must visibly change the motion, to prove the system is real. Show the frame rate, plus a live readout for the selected character: steps per second, planted-foot drift in cm and maximum knee flexion.

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

ADD A PROCEDURAL WALK, RUN AND FIGHT-STANCE ANIMATION to every humanoid, driven by the distance actually travelled, with foot-pinning leg IK. No animation files, no jitter, no sliding feet. Build it exactly as described; every number below fixed a real defect.

WHAT IT MUST DO
- Feet plant and stay planted: a planted foot drifts under 1 cm (median) while the body moves over it.
- Legs move at ground speed. Stride length matches leg length. No skating, no running in place.
- The body does not shake, vibrate or pop. Knees never snap at full extension.
- Standing, walking, running and fighting stance are one continuous system that blends smoothly (about 6 per second), with idle breathing and weight shifts when still.

1. PHASE FROM DISTANCE. Gait phase advances by speed * dt / stride, never by clock time. Speed is the distance the character actually moved this frame divided by dt (negative when walking backwards), smoothed with k = 1 - exp(-5 dt). Step length for a leg of 0.85 m: at 1.35 m/s or less, max(0.5, speed / 1.6); from 1.35 to 2.3 m/s, max(0.78, speed / 1.95); toward a run (above 3 m/s), speed / 2.75. Targets: party walk 1.6 m/s, step about 0.82 m, cadence 1.9 to 2.0 steps per second; run 2.8 steps per second; fighting footwork 1.6 steps per second. A step above about 0.9 of the leg length is wrong (0.95 m on a 0.85 m leg looked like overstriding). Scale everything with the character's leg length.

2. LEG TARGETS (foot position along the walking direction, relative to the hip). Stance: the foot moves backward at exactly ground speed, so it stays put in the world. The foot lands about 0.38 of a step ahead of the hip and leaves about 0.62 of a step behind (not symmetric). Swing: carry the foot from its release point to 0.38 step ahead with a smoothstep. A trailing planted foot lifts its heel (up to 11 cm, toe stays down; toe pitch = asin(lift / 0.2), capped at 0.6 rad) before it releases. Release a plant when the foot is 0.56 m behind the hip while walking (0.33 m when standing or fighting), when the leg is 90 percent extended or folded too far, or when the foot's yaw is more than 0.66 rad off the body's facing.

3. SWING KNEE. Swing knee flex peaks at about 0.62 rad at walking speed and 1.05 rad at a run, blended with smoothstep(2.2, 3.2, speed). That is about 40 percent less than the run value at walking speed; full flex at a walk kicked the shin up like a jog. In fighting footwork use 0.5.

4. HIPS. Do not hold the hips low for the whole cycle (that made a permanent deep crouch). The hips drop only as far as the front leg needs at heel strike: need = L - sqrt(L*L - reach*reach), reach = min(0.72 L, step * 0.38). Add a bob of 2.5 cm at a walk to 4 cm at a run, exactly two bobs per stride, shaped bob * (1 + cos(2 * phase)), so the hips rise to about twice the bob at mid-stance. Add pelvis yaw 0.05 rad and roll 0.03 rad with the legs and a sideways sway of 1.4 cm.

5. UPPER BODY. Arms swing opposite to the legs: shoulder swing 0.42 * (0.6 + 0.4 * speed factor) rad, forearm bent 0.18 to 0.40 rad, faded in between 1.0 and 2.0 m/s. Spine counter-twists 0.13 rad and chest 0.10 to 0.17 rad against the pelvis; the head counter-rotates so it stays level. Forward lean is about 0.05 rad at a walk and 0.2 rad at a run: walking lean is about half the running lean. Cloaks, hair, lanterns and tails follow on damped springs (stiffness 40 to 60, damping 6 to 13) driven by acceleration and the gait.

6. LEG IK, every frame, after the pose is built. Two-bone solver with the knee on a pole vector (the pelvis's forward direction flattened to the ground), knee flexion limited to 5 to 120 degrees, and the target distance clamped to the leg's minimum and maximum reach (the maximum is at 5 degrees of flexion, never fully straight). Planted foot: store its world position and yaw when it lands, keep it flat on the ground, and blend the pin in and out over about 120 ms (weight rate 8.3 per second, smoothstep). Lower the hips (smoothed with exp(-5 dt)) only as much as the planted leg needs to reach. During swing the animated pose supplies the foot target.

7. VIBRATION RULES. All shake came from per-frame noise in the pose. Never add random or high-frequency noise to the root, hips, camera or knees.
   - Knee angle goes through a critically damped spring (omega 34) and is capped at 400 degrees per second, which is no more than 6.7 degrees per frame at 60 fps.
   - Smooth the final hip height with two cascaded exponential filters, rate 28 per second.
   - Root and camera motion above 6 Hz must stay under 1 mm.
   - The spring cap plus a valid IK solve near full extension (the 5-degree minimum flex) stops the knee pop.
   - On the first frame, or after a pause longer than 0.1 s, reset the springs to their targets instead of integrating.

8. FIGHT AND ACTIONS. Attacks are short key-framed clips (windup, hit at a fixed fraction, recover) of bone Euler angles plus a root offset, eased in over the first 7 percent and out over the last 16 percent, layered on top of the walk. The upper body turns before the hips and feet. Combat footwork uses the same planting system.

BROWSER SAFETY: Windows Chrome runs WebGL through D3D11. Pose and skin on the CPU in JavaScript, or in a vertex shader with no texture reads and no dynamic indexing of uniform arrays. Vertex attributes are floats only. Constant loop bounds.

PERFORMANCE: fill the window, resize cleanly, 60 fps on a modern GPU and zero console errors. Use three.js, delivered inside the single HTML file with no network fetches. Windows safety (Chrome on Direct3D): sample textures only in fragment shaders, float vertex attributes only, no dynamic indexing of uniform arrays in vertex shaders, constant loop bounds, internal render resolution capped at about 2 megapixels, and a visible message if the WebGL context is lost.

IT MUST RUN THE FIRST TIME: the page has to work correctly the moment it is opened, with zero console errors, no blank screen and no frozen frame.
- One script in strict order: constants, helpers, classes, scene setup, and the animation loop started last. Never call a function or read a variable before it is defined.
- Load three.js from one pinned CDN version as an ES module with an import map, and load every addon from that same version. Use only APIs that exist in that version.
- Define every name once and use it with exactly the same spelling everywhere. Keep settings and shared state in one object instead of loose globals.
- Guard the maths that breaks rigs and IK: clamp every acos and asin input to [-1, 1], never divide by a length that can be zero, normalise only non-zero vectors, and if a value ever becomes NaN, keep the last good value so one bad frame can never poison a pose.
- Clamp the frame delta (for example to 1/30 s), so a hidden tab or a stall never teleports anyone.
- Create geometry, materials and textures once at startup, never per frame.
- Wrap startup in try/catch and show any error as readable text on the page instead of a blank screen.
- Deliver the whole file: no placeholders, no TODOs, no "rest of the code here", no shortened functions.
