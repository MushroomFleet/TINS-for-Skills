# WIPEOUT 2097: Complete Reference Guide for a Spiritual Successor

The 1996 anti-gravity racer WIPEOUT 2097 (WIPEOUT XL in North America) represents the pinnacle of the genre, combining precise physics, The Designers Republic's iconic visual identity, and groundbreaking electronic music integration. This comprehensive reference provides everything needed to recreate that experience using modern web technologies like React, Three.js, Drei, and Fabric.js.

## The physics model that defined anti-gravity racing

WIPEOUT 2097's ships don't drive—they **float on invisible springs** above the track surface, creating a distinctive hovercraft-like feel that emphasizes momentum over grip. The original PlayStation implementation used fixed-point math at 30fps with a spring equation calculating flotation force based on track polygon normals and ship height (`F=k/z`).

**Core handling characteristics** distinguish WIPEOUT from conventional racers. Ships have significant inertia requiring corner anticipation, with steering accomplished through a combination of directional input and **airbrakes** (L2/R2). The left airbrake swings the rear end left while veering the craft leftward; the right does the opposite. This creates a distinctive drifting mechanic where players must **turn before the corner** and airbrake to arrive facing the correct direction. Pitch control (D-pad up/down) affects cornering tightness and landing behavior after jumps.

The four speed classes escalate dramatically: **Vector** (beginner), **Venom** (intermediate), **Rapier** (hard), and **Phantom** (expert—50% faster than Rapier, reaching **450+ km/h**). Each class adds one lap to races and demands increasingly precise airbrake technique.

### Ship statistics and team differences

Five racing teams offer distinct handling profiles on a 1-10 scale:

| Team | Thrust | Top Speed | Turning | Shield | Character |
|------|--------|-----------|---------|--------|-----------|
| **FEISAR** | 6 | 3 | 7 | 6 | Beginner-friendly, most maneuverable |
| **AG Systems** | 7 | 5 | 6 | 3 | Best acceleration, weak shields |
| **Auricom** | 5 | 6 | 5 | 5 | Balanced all-rounder |
| **Qirex** | 4 | 7 | 3 | 7 | Fastest but hardest to control |
| **Piranha** | 10 | 10 | 10 | 10 | Unlockable perfect stats, no weapons |

The **shield/HP system** (introduced in 2097) allows ship destruction. Shields deplete from wall impacts (slight drain), ship collisions (slight drain), and weapon hits (significant drain). At 25% capacity, a voice warns "Shield energy low." At zero, the craft explodes. **Pit lanes**—red-colored track sections—restore shield energy at the cost of precious seconds, introducing strategic depth absent from the original WIPEOUT.

### Collision behavior that defined the feel

Unlike the original WIPEOUT where ships stopped dead against walls, 2097 introduced **wall scraping with sparks**—ships bounce off rather than halting, losing speed and shield energy proportionally to impact severity. This single change transformed the game from frustrating to accessible. Landing nose-down after jumps causes "bottoming out" with bouncing and severe speed loss; raising the nose softens landings.

## Visual identity crafted by The Designers Republic

Sheffield-based design agency **The Designers Republic** (tDR), founded by Ian Anderson in 1986, created WIPEOUT's entire visual universe. Their "maximum-minimalist" approach combined Japanese anime influences, anti-establishment irony, and slogans like "Work Buy Consume Die" to craft a believable dystopian future.

**Every team functioned as a complete branding exercise**, with logos, typography, trackside advertising, and fictional corporate sponsors forming a cohesive world. The WIPEOUT logo itself uses partial "8" glyphs from the Eurostile typeface—the established sci-fi font since the 1960s—with the apostrophe and quotation marks denoting minutes and seconds used in racing.

### Technical specifications by platform

The PlayStation version ran at **320×240** resolution at 30fps (NTSC), using heavy full-screen dithering for color depth on the limited hardware. The Saturn version offered slightly lower framerates but **longer draw distance** and less dithering. The PC version supported 3D acceleration with higher resolutions and smoother framerates, though with different lighting characteristics.

Textures used the PlayStation's native **TIM format** with 16-bit direct colors or 4/8-bit palette indices. The track renderer employed a **level-of-detail system** subdividing faces up to 4×4 quads when near the camera, compensating for the PSX's lack of perspective correction. Each tile measured 32×32 pixels with LOD variants at 4×4, 2×2, and 1×1.

### The cyberpunk color philosophy

The color palette established the template for cyberpunk gaming aesthetics:

- **Neon highlights** against dark backgrounds
- **Electric blues and cyans** representing technology and speed
- **Hot pinks and magentas** signaling energy and danger
- **Bright yellows** for warnings and synthetic energy
- **Deep purples** creating digital depth
- **Near-black backgrounds** for dramatic contrast

Team-specific palettes included FEISAR's blue/gold, AG Systems' red/white with Japanese katakana, Qirex's distinctive purple/magenta with faux-Cyrillic typography, and Piranha's aggressive red/black.

### HUD design principles

The interface maintained a **minimalist, clinical aesthetic** with clean Eurostile typography throughout. Essential elements included a numeric speed readout, position/lap counter, purple shield energy bar, current weapon symbol (top-center), and lap times. Voice announcements provided tactical warnings like "Missile lock" and "Shield energy low."

## Track design that created the sensation of speed

WIPEOUT 2097's eight tracks demonstrate masterful level design, progressing from accessible introductions to demanding expert challenges while maintaining the signature sense of extreme velocity.

### Design philosophy and element vocabulary

Tracks were designed for **continuous momentum with strategic braking points**, using width variation to create rhythm—narrow sections increase tension and punish mistakes while wide sections offer recovery. **Elevation changes** create both visual drama and mechanical challenge: peaks cause "bottoming out" without nose adjustment, troughs demand nose-up to prevent scraping, and dramatic drops combine speed with technical difficulty.

**Tunnel sections** reduce visibility and force memorization, increasing tension and rewarding familiarity. **Jump sections** require pitch management—raising the nose softens landings while lowering it reduces height (critical for entering tunnels). Some jumps demand mid-air turning, such as Spilskinanke's infamous 90-degree aerial corner.

**Speed pads** (blue arrow blocks) provide small boosts when crossed, while animated roadside objects, team logos, and Red Bull advertising create visual reference points that enhance perceived velocity.

### The iconic eight tracks

**TALON'S REACH** (Canada, 3.2km, Vector class) serves as the perfect introduction—a fully indoor industrial complex with yellow steam jets and forgiving corners that teach airbrake fundamentals without overwhelming new players.

**SAGARMATHA** (Nepal, 4.3km, Vector class) introduces elevation in a snowy Himalayan setting with 158m of height variation, teaching chicane navigation and blind turn handling through slow-moving snow particles and mountain-carved sections.

**VALPARAISO** (Chile, 3.9km, Venom class) presents 220m of elevation change through jungle temples, with a checkered track pattern that deliberately obscures corner apexes—testing genuine airbrake proficiency.

**PHENITIA PARK** (Germany, 3.8km, Venom class) adds urban industrial complexity with jumps leading directly into corners and the introduction of turbo shortcut possibilities through buildings.

**GARE D'EUROPA** (France, 3.5km, Rapier class) stands as arguably the series' most famous track—a disused metro system with **179m of elevation change** (highest in the game), atmospheric lightning and rain, and the longest straight section culminating in a massive undulating jump. Target lap time: **~41 seconds** at Phantom class.

**ODESSA KEYS** (Ukraine, 4.4km, Rapier class) earned the community's "most hated" designation for its water-suspended sections with sharp corners in dark areas and a final right-hander that's impossible at full speed in heavier craft.

**VOSTOK ISLAND** (Pacific, 5.1km, Phantom class) and **SPILSKINANKE** (California, 4.0km, Phantom class) unlock after completing the Phantom Challenge. Spilskinanke—an anagram of "Snake Plissken"—is the game's hardest track, featuring post-earthquake urban ruins, near-total darkness, blind turns, and the notorious 90-degree aerial turn.

### Pit stop integration

Pit lanes appear once per track, typically near the start/finish line but sometimes requiring alternative route choices. The red-colored sections provide electrical energy regeneration visuals while forcing a strategic decision: seconds lost versus shield recovery.

## Anti-gravity physics for modern implementation

The 2022 source code leak and subsequent community analysis revealed the specific techniques that created WIPEOUT's distinctive feel—knowledge essential for accurate recreation.

### The four-point raycast hover system

Modern implementations use **four raycasts from ship corners** detecting track surface distance, with spring-damper forces calculated per point:

```javascript
// Force calculation per corner
float hoverForce = ((hoverDistance - restingHeight) / Time.deltaTime) * 
                   (-0.2f * (restingHeight - hoverDistance));
hoverForce -= hoverDamping * rigidbody.velocity.y;
```

**Key parameters** for authentic feel: physics updates at **60-100Hz minimum** (50Hz absolute minimum for single raycast), angular damping around **20** for smooth rotation, and separate gravity values—lighter when grounded, heavier when airborne, lerping between states during transitions.

### Separating physics from visuals

Professional implementations use an **invisible chassis** handling physics/raycasting, connected via spring-damper to a **visible body mesh** that can wobble independently. This allows identical physics with different visual behaviors. Ship banking during turns applies **only to the mesh's local Z rotation**, not the physics object—creating visual tilt without affecting handling calculations.

### Wall collision approach

Angling wall colliders **inward by 10-15 degrees** applies downward force on contact, preventing ships from climbing walls. Separate collision meshes for walls and floors (simpler geometry than visual meshes) improve performance and behavior consistency. Optional side raycasts can apply repulsion force before contact for smoother wall proximity handling.

## Series evolution and what made 2097 definitive

Understanding why 2097 achieved iconic status guides which elements demand faithful recreation versus modernization.

### The critical improvements from WIPEOUT 1

The original 1995 WIPEOUT suffered from **ships halting abruptly** on wall contact and weapons that only slowed opponents. 2097 fixed both issues: wall scraping replaced dead stops, and the new shield system enabled ship destruction, adding stakes and strategy. The controls became "much more refined, fairer, and easier to master" while maintaining depth.

### Why IGN called it "stratospherically good"

GamePro awarded **perfect 5/5 scores** in all categories. Electronic Gaming Monthly named it **Best Music of 1996**. The game arrived at the perfect cultural moment—Christmas 1996 during the transition from 16-bit to 32-bit gaming. Sony's marketing genius placed PlayStation consoles in nightclubs; game discs doubled as audio CDs; branded clubwear appeared in rave culture.

Co-creator Nick Burcombe's reflection: "It is a game with a soul and I think people get that. The ideas, the graphic design, the music and the PlayStation all combined to make it perfect for that moment in time."

### The music integration that defined gaming's relationship with electronic culture

WIPEOUT pioneered **licensed electronic music** in games—enabled by PlayStation's CD audio capability. The 2097 soundtrack featured The Chemical Brothers ("Loops of Fury"), The Prodigy ("Firestarter" instrumental), Underworld, Future Sound of London, Photek, and Fluke. In-house composer CoLD SToRAGE (Tim Wright) created additional tracks in his "freezing cold" Liverpool warehouse studio, "fuelled by Red Bull, pizza and long summer evenings."

The soundtrack album reached **#16 on the UK Compilations Chart**. The game "exposed millions to underground club and rave music" and "brought the nightclub experience into bedrooms across the globe."

### The Designers Republic departure's impact

After working on WIPEOUT, 2097, and WIPEOUT 3, tDR was asked to **pitch competitively** against other agencies for Fusion (2002). Ian Anderson's response: "We do not pitch for jobs, we either do them or we do not." The resulting Fusion was immediately criticized as "soul-less" with "fairly lacklustre" UI—demonstrating how essential tDR's design was to the franchise's identity.

## Technical implementation for Three.js and web technologies

### Core architecture pattern

```javascript
// Main game loop structure
class GameApp {
    run() {
        this.update(this.deltaTime);
        this.renderer.render(this.scene, this.camera);
        requestAnimationFrame(() => this.run());
    }
}
```

### Physics engine recommendations

**Cannon.js** offers the best Three.js compatibility with RayCastVehicle class for hover simulation. **Rapier.js** provides WASM-based performance improvements. Key parameters to tune: suspension stiffness (high for responsive handling), center of mass position (affects handling dramatically), angular drag (critical for stability), and hover height with damping ratios.

### Camera system for speed sensation

WIPEOUT HD's Icarus ship used camera offset `(0, -10.8, 3.5)` with look-at point `(0, -4, 25)` ahead. Position lerping creates the trailing effect:

```javascript
camera.position.lerp(targetPosition.add(cameraOffset), 0.1);
camera.lookAt(ship.position.clone().add(lookAtOffset));
```

### Track geometry approaches

**Spline extrusion** works well for procedural generation—define control points, interpolate with Catmull-Rom splines, extrude cross-section profiles. For track editors, separate the **collision mesh** (simplified geometry) from the **visual mesh** (detailed). Track validation should check self-intersections, minimum width, and driveable racing lines.

**Automatic banking detection** measures curvature at each spline segment and applies proportional banking, with parametric override for manual adjustment.

### Time tracking and ghost replay

Record timestamp, position (x, y, z), and quaternion rotation every physics frame. Compress to **~3-4 KB per run** for storage. Playback interpolates between frames:

```javascript
const t = (currentTime - frame1.timestamp) / (frame2.timestamp - frame1.timestamp);
ghostPosition = lerp(frame1.position, frame2.position, t);
```

**Sector timing** uses invisible trigger volumes at boundaries, calculating deltas against best/comparison laps.

## Essential resources for development

**Open source implementations** provide invaluable reference: the [WIPEOUT Rewrite](https://github.com/phoboslab/wipeout-rewrite) compiles to WASM and runs in browsers; [WIPEOUT Phantom Edition](https://github.com/wipeout-phantom-edition/wipeout-phantom-edition) offers enhanced source port with modern features; BallisticNG's [Unity tools](https://github.com/Neognosis-Workflow/BallisticNG-Unity-Tools) demonstrate professional track editor implementation.

For Three.js specifically, examples like [react-threejs-car-racing](https://github.com/sctlcd/react-threejs-car-racing) demonstrate React Three Fiber with Cannon.js integration—directly applicable to your React/Three.js/Drei stack.

## The formula that must be preserved

Fan consensus and developer reflection converge on these essential elements: **pitch control as fundamental technique**, the floaty anti-gravity feel with precise airbrakes, wall scraping rather than dead stops, the **Designers Republic aesthetic** with coherent fictional branding, electronic music matching gameplay tempo, and racing-first balance where speed skill matters more than weapon spam.

WIPEOUT 2097's genius lay in fixing frustrations while maintaining depth—accessible to newcomers yet demanding years to master. Nick Burcombe called it "the game with a soul," and recreating that soul requires understanding not just the mechanics, but the philosophy: anti-gravity racing should feel like piloting a hovercraft at impossible speeds through a believable dystopian future, where every corner rewards anticipation and every straightaway delivers the pure rush of velocity.