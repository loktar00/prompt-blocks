# Prompt

The full prompt for the fog test scene, including the reusable fog recipe. Paste it into a coding model or agent as one message.

```text
Create one complete, self-contained HTML file for FOG TEST, a small real-time three.js scene whose only purpose is to prove a volumetric fog effect and show that it REACTS to an object moving through it. Keep the scene tiny so all the effort goes into the fog.

THE SCENE, deliberately small:
- A square stone floor about 40 units across with a subtle tiled texture drawn in code, floating on a dark background (#16171b).
- Four simple props standing on it: a stone pillar, a low wall with a doorway-sized gap, a sphere on a plinth and a small leafless tree. Nothing else.
- ONE CUBE, about 3 units across, in a strong warm colour (orange) so it reads clearly. It hovers just above the floor.
- One low sun, about 25 degrees above the horizon, casting real shadows from a shadow map. A dim cool ambient.
- Camera: a fixed three-quarter view from above that frames the whole floor. Mouse drag orbits, the wheel zooms, and it never goes below the floor.

THE CUBE, how it moves:
- By default it glides by itself along a slow figure-of-eight path that passes through the wall gap, around the pillar and through the thickest fog, on a loop of about 20 seconds, so the page shows the effect with no input.
- The user can take over: W A S D or the arrow keys move it on the floor plane, Q and E lower and raise it, and it can also be dragged with the mouse (drag the cube itself; dragging empty space orbits the camera). After 5 seconds without input it eases back onto its path.

THE FOG, the point of the piece. Build it exactly as the recipe below describes, with these coverage shapes: a ground layer about 2 units deep over the whole floor, a thicker bank about 6 units tall drifting slowly across the middle, and thin haze everywhere up to 14 units.

THE FOG MUST VISIBLY REACT TO THE CUBE, in all four of these ways:
1. OCCLUSION: fog in front of the cube veils it and fog behind it is hidden by it, correctly per pixel (the march stops at the scene depth).
2. SHADOW IN THE AIR: the cube is in the shadow map, so it carves a dark SHADOW VOLUME through the lit fog, a visible dark beam leaving the cube away from the sun, which swings as the cube moves. The pillar, wall, sphere and tree do the same, and light shafts come through the wall gap.
3. DISPLACEMENT: the cube pushes fog aside. Subtract density inside a soft sphere around the cube (radius about 1.6 times the cube) so the cube always sits in a clear pocket.
4. A WAKE: keep the cube's recent positions (about 24 samples, one every 0.12 s) in a uniform array and subtract a little density around each, weaker and wider with age, so a clear tunnel trails behind the moving cube and slowly fills back in over about 3 seconds. Add a small swirl: offset the noise lookup near the trail sideways, so the tunnel's edges curl.

A SMALL PANEL, top-left, plain and tidy, with the controls listed in the fog recipe below plus sun height, so each reaction can be switched off and on to prove it is real. Show the frame rate.

ADD REAL VOLUMETRIC GROUND FOG that drifts, catches light and is stirred by moving objects. Build it exactly as described; every part matters. It is one extra render pass on top of the finished scene, and it must be physically honest: fog is density, and nothing else.

WHAT IT MUST DO
- A low carpet of fog hugging the ground (default about one object-height deep), with a rolling, uneven top, drifting with the wind.
- Moving objects plough through it: they sit in a clear pocket, push a bow wave ahead, drag the fog along and leave a curling wake that slowly fills back in.
- Everything in the scene shadows it: objects carve dark shadow beams through lit fog, and light comes through gaps as shafts.
- Fog in front of an object veils it; fog behind is hidden by it, correct per pixel.
- With fog density at 0 the picture is pixel-identical to the scene without the fog pass. No glow or tint may survive.

1. ONE 3D NOISE TEXTURE, built once in code at load: 64 x 64 x 64, single channel, tileable, REPEAT wrap, linear filtering. Each voxel = 72 % four-octave value-noise fBm + 28 % inverted Worley (cellular) noise, contrast-stretched around the middle. Every fog lookup samples this texture; never compute noise per pixel in the shader.

2. A FLOW FIELD THE OBJECTS STIR. A small ping-pong render target (128 x 128, half-float, two channels) laid over the ground in world XZ, storing a 2D velocity. Every frame, in one full-screen pass:
   - advect it by itself (semi-Lagrangian: look up where each texel came from, one sample);
   - blend in a little of its four neighbours (about 12 %) as diffusion, and damp it (about 0.97 per frame, scaled by dt) so motion dies away over a few seconds;
   - STAMP each moving object's velocity into a soft disc about two object-widths across, with a forward push in the front half and a sideways curl of opposite sign on each flank, so vortices peel off behind it.

3. THE DENSITY FUNCTION, coverage times noise, bent by the flow:
   - sample the 3D noise at three or four scales that scroll at different speeds and slightly different directions with the wind, so the fog rolls and shears; the large shapes travel about 1.5 to 2 units per second at the default wind; let the medium scale WARP the large one's lookup position;
   - SQUEEZE the fine scales along the wind direction, so detail streaks into wisps, not round puffs;
   - OFFSET the noise lookup position by the flow field's velocity (scaled by a "stir strength", fading out with height above the objects). This is what makes the fog visibly move with the objects;
   - coverage: a ground layer whose ceiling is "fog height" times a noisy factor (about 0.66 to 1.0), so the top is tufted, plus optional slow-drifting swells up to about 1.6 times the fog height; a "height falloff" control blends between an even column and density that thins quickly with height;
   - density = clamp((noise - (1 - coverage)) * 1.9, 0, 1); then ERODE edges with the fine noise so thin edges become threads and thick cores stay solid;
   - then, around each object: subtract a soft sphere (radius about 1.6 times the object) for the clear pocket; subtract along its recent positions (keep about 24 samples, 0.12 s apart, weaker and wider with age) for the wake; ADD a little density ahead and to the sides in proportion to its speed for the bow wave;
   - all of these change DENSITY only.

4. A HALF-RESOLUTION RAYMARCH PASS. After the opaque scene is drawn into a target with a depth texture, draw one full-screen pass into a half-width, half-height half-float target. For each pixel march the camera ray through the fog slab only (from the ground to about 1.65 times the fog height):
   - at most 64 steps, with the step length scaled to the slab (shorter slab, finer steps);
   - start with a per-pixel jitter of one step (interleaved gradient noise), so banding becomes fine grain;
   - STOP at the scene depth from the depth texture; stop early when transmittance falls under about 0.02;
   - per step: sigma = density * fogDensity; a = 1 - exp(-sigma * step); light = sun colour * shadowMap(position) * selfShadow + ambient; colour += T * a * light; T *= 1 - a;
   - shadowMap(position) is the scene's ordinary shadow map sampled at the step position, so every object casts a shadow volume through the fog;
   - selfShadow = exp(-density sampled once a short way toward the sun * 6);
   - optionally a very thin "haze" above the fog (its own slider, scaled by fogDensity too), lit the same way, so shafts and shadow beams show above the carpet.
   Output premultiplied colour and the remaining transmittance.

5. COMPOSITE at full resolution with a DEPTH-AWARE upsample: take a 3 x 3 neighbourhood of the half-res fog, weight each tap by how close its depth is to this pixel's depth (centre tap weighted 3x), then final = scene * fogTransmittance + fogColour. Silhouettes stay crisp, with no halos.

6. HONESTY RULE. Light enters the picture ONLY through the line in step 4. No additive glows, no tints, no screen-space "fog colour" washes. The bow wave, pocket and wake are density changes.

7. CONTROLS, in a small panel: fog density, fog height, height falloff, haze, wind speed, stir strength, wake strength, and toggles for "shadows in fog", "displacement", "wake" and "flow field", so each behaviour can be switched off to prove it is real.

BROWSER SAFETY: sample textures only in fragment shaders; use textureLod with explicit LOD 0 inside loops and branches; keep loop bounds constant (64 march steps, 24 wake samples) and exit with break; cap the internal render resolution at about 2 megapixels.

Windows safety (Chrome on Direct3D): sample textures only in fragment shaders; use textureLod with explicit LOD 0 for every lookup inside a loop or branch; keep the march loop bound a constant (64) and exit with break; keep the wake array a fixed size (24) with a constant loop; cap the internal render resolution at about 2 megapixels; show a visible message if the WebGL context is lost.

PERFORMANCE: fill the window, resize cleanly, 60 fps on a modern GPU and zero console errors. Use three.js, delivered inside the single HTML file with no network fetches.

IT MUST RUN THE FIRST TIME: the page has to work correctly the moment it is opened, with zero console errors, no blank screen and no frozen frame.
- One script in strict order: constants, helpers, classes, scene setup, and the animation loop started last. Never call a function or read a variable before it is defined.
- Load three.js from one pinned CDN version as an ES module with an import map, and load every addon from that same version. Use only APIs that exist in that version.
- Define every name once and use it with exactly the same spelling everywhere. Keep settings and shared state in one object instead of loose globals.
- Guard the maths that breaks rigs and IK: clamp every acos and asin input to [-1, 1], never divide by a length that can be zero, normalise only non-zero vectors, and if a value ever becomes NaN, keep the last good value so one bad frame can never poison a pose.
- Clamp the frame delta (for example to 1/30 s), so a hidden tab or a stall never teleports anyone.
- Create geometry, materials and textures once at startup, never per frame.
- Wrap startup in try/catch and show any error as readable text on the page instead of a blank screen.
- Deliver the whole file: no placeholders, no TODOs, no "rest of the code here", no shortened functions.

```
