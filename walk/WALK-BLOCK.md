# Walk block

Procedural walk, run and fight-stance animation with foot-pinning leg IK, driven by distance travelled. Use it together with the character model block.

```text
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

```
