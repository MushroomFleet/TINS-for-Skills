# Sky Drift Adventures

## Description

Sky Drift Adventures is a side-scrolling platformer game inspired by classic Game Boy mechanics, featuring a fluffy cloud character navigating through atmospheric levels. The player controls Nimbus, a cheerful cloud who can inhale wind currents, float through the sky, and use air projectiles to defeat corrupted weather spirits that have disrupted the peaceful Sky Realm.

The game implements classic platformer mechanics including air inhale/exhale for combat and flight, slide attacks, and boss battles. Built as a React single-page application using pure SVG graphics with minimal 2-frame animations, the game prioritizes simplicity and nostalgia while maintaining smooth gameplay through component-based architecture.

Players progress through five distinct atmospheric zones, each with unique visual themes and enemy patterns, culminating in a final confrontation with Lord Cumulus who has hoarded all the Sky Realm's precipitation.

## Functionality

### Core Gameplay Mechanics

#### Movement System
- **Walking:** Arrow keys (Left/Right) move Nimbus horizontally at 3 pixels per frame
- **Running:** Hold Shift while walking to move at 5 pixels per frame
- **Jump:** Spacebar causes Nimbus to jump with initial velocity of -12 pixels/frame, affected by gravity (0.6 pixels/frame²)
- **Platform Collision:** Nimbus lands on platforms when descending and overlapping platform hitboxes
- **Screen Bounds:** Nimbus cannot move beyond left/right screen edges (0 to 800 pixels)

#### Inhale/Exhale System
- **Inhale Activation:** Press and hold 'Z' key to enter inhale mode
- **Inhale Range:** Creates a 60-pixel cone in front of Nimbus that pulls enemies within range
- **Pull Force:** Enemies within cone move toward Nimbus at 4 pixels/frame
- **Capture:** When enemy reaches Nimbus's center (within 20 pixels), enemy is stored internally
- **Visual Feedback:** Nimbus's mouth expands to 1.3x scale during inhale, shows swirling air particles (3-5 rotating dots in cone area)
- **Swallow:** Release 'Z' while holding enemy to swallow (enemy disappears, no effect in this version - no ability copy)
- **Spit Projectile:** Press 'X' while holding enemy to spit enemy as projectile traveling at 8 pixels/frame in facing direction
- **Projectile Collision:** Spit projectile deals 2 damage to enemies it hits, destroys on impact
- **Air Puff Attack:** Press 'X' without holding enemy to shoot star-shaped air puff projectile (1 damage, travels 6 pixels/frame for 80 pixels maximum range)

#### Flight System
- **Inflate:** Press Spacebar while airborne (after jump or falling) to inflate with air
- **Inflated State:** Nimbus visually expands to 1.5x size, gravity reduces to 0.1 pixels/frame²
- **Flying Movement:** While inflated, Arrow keys control both horizontal (3 pixels/frame) and vertical movement (Up: -2 pixels/frame, Down: exhale)
- **Flight Duration:** Unlimited - player can stay airborne indefinitely
- **Exhale/Descend:** Press Down arrow to deflate and return to normal gravity
- **Cannot Inflate:** If already on ground and not jumping, must jump first to enter flight

#### Slide Attack
- **Activation:** While running (Shift + Arrow), press Down arrow
- **Slide Behavior:** Nimbus enters crouched state, moves forward at 6 pixels/frame for 30 frames (1 second at 30 FPS)
- **Hitbox:** While sliding, Nimbus height reduces to 50% normal height, width stays same
- **Damage:** Slide deals 1 damage to enemies contacted during slide
- **Slide End:** After 30 frames, Nimbus returns to standing state at current position
- **Cannot Slide:** If moving left, if airborne, or if already sliding

### Enemy System

#### Enemy Types
All enemies have 2-frame idle animations (alternating every 15 frames):

1. **Breeze Wisp** (basic flying enemy)
   - Health: 2 HP
   - Behavior: Flies horizontally left-right in 150-pixel patrol range, 2 pixels/frame
   - Visual: Small swirling wind spiral (30x30 pixels), alternates between tight/loose spiral
   - Collision Damage: 1 HP to player

2. **Drizzle Sprite** (walking enemy)
   - Health: 2 HP
   - Behavior: Walks on platforms left-right in 200-pixel patrol range, 1.5 pixels/frame
   - Visual: Small rain droplet with face (25x35 pixels), alternates between round/teardrop shape
   - Collision Damage: 1 HP to player

3. **Thunder Spark** (bouncing enemy)
   - Health: 3 HP
   - Behavior: Bounces in arc pattern - jumps every 60 frames with velocity -10, follows gravity
   - Visual: Yellow lightning bolt (20x40 pixels), alternates between jagged/straight
   - Collision Damage: 2 HP to player

4. **Frost Puff** (floating obstacle)
   - Health: 4 HP
   - Behavior: Floats vertically up-down in 100-pixel range, 1 pixel/frame, reverses at bounds
   - Visual: Snowflake (35x35 pixels), alternates between 6-point/8-point star
   - Collision Damage: 1 HP to player

#### Enemy Behavior Rules
- Enemies reverse direction when reaching patrol range bounds
- Enemies cannot move through walls/platforms
- When defeated (health reaches 0), enemy plays a 10-frame puff animation then disappears
- Enemies respawn at original positions when player dies or exits/re-enters level
- Maximum 8 enemies visible on screen simultaneously

### Boss System

Each of the 5 levels ends with a boss battle in a dedicated arena room (800x600 pixels):

#### Level 1 Boss: "Gale Commander"
- Health: 20 HP
- Phase 1 (20-11 HP):
  - Behavior: Floats horizontally across screen top (y=100), shoots 3 air puffs downward every 90 frames
  - Air Puffs: Travel at 3 pixels/frame downward, deal 2 damage
- Phase 2 (10-0 HP):
  - Behavior: Moves faster (5 pixels/frame), creates 2 Breeze Wisps every 120 frames
  - Wisps: Standard enemy behavior but only exist for 180 frames
- Visual: Large swirling vortex (100x100 pixels), spins continuously, changes color intensity by phase

#### Level 2 Boss: "Monsoon Sentinel"
- Health: 25 HP
- Phase 1: Ground pound attacks (jumps to y=50, slams down creating shockwave every 120 frames)
- Phase 2: Summons 3 Drizzle Sprites, throws water orb projectiles (6 pixels/frame, 3 damage)
- Visual: Giant rain cloud with angry face (120x80 pixels)

#### Level 3 Boss: "Volt Tyrant"
- Health: 30 HP
- Phase 1: Teleports to random positions every 90 frames, shoots lightning in 8 directions
- Phase 2: Summons Thunder Sparks, increases teleport frequency to 60 frames
- Visual: Concentrated storm cell (90x90 pixels), crackling electricity

#### Level 4 Boss: "Blizzard Baron"
- Health: 35 HP
- Phase 1: Creates ice platform walls that move inward from edges, shoots ice spears horizontally
- Phase 2: Platforms move faster, summons Frost Puffs, shoots in multiple directions
- Visual: Crystalline ice formation (110x90 pixels)

#### Level 5 Final Boss: "Lord Cumulus"
- Health: 50 HP
- Phase 1 (50-35 HP): Slow movement, shoots single projectile type (air puffs)
- Phase 2 (34-20 HP): Faster movement, combines two projectile types
- Phase 3 (19-0 HP): Maximum speed, all projectile types, summons enemies from previous bosses
- Visual: Massive dark storm cloud (150x120 pixels), lightning crackling within
- Victory: Defeats Lord Cumulus, sky clears, peaceful weather returns (ending scene)

#### Boss Battle Rules
- Boss arena locks (cannot leave room until boss defeated)
- Boss has invincibility frames: 15 frames after taking damage (flashes white/transparent)
- Player death in boss room restarts boss fight from beginning (boss full health)
- Defeated bosses do not respawn; level exit opens

### Level Design

#### Level Structure
Each level consists of:
- **Level Width:** 3200 pixels (4 screens wide, 800 pixels per screen)
- **Level Height:** 600 pixels (single screen height)
- **Camera:** Follows player horizontally, centered on player (player at x=400 on screen)
- **Platforms:** 8-15 rectangular platforms per level, varying widths (60-300 pixels), heights (20-40 pixels)
- **Enemy Placement:** 12-20 enemies distributed across level
- **Collectibles:** None (simplified from original)
- **Checkpoints:** None (simplified from original)
- **End Portal:** At rightmost edge (x=3150), teleports to boss arena when touched

#### Level Themes

**Level 1: "Gentle Breeze Plains"**
- Color Palette: Light blue sky (#87CEEB), white clouds (#FFFFFF), soft green platforms (#90EE90)
- Enemies: Primarily Breeze Wisps (10) and Drizzle Sprites (6)
- Platform Layout: Gentle slopes, wide platforms suitable for learning mechanics
- Background: Distant mountain silhouettes (#B0C4DE)

**Level 2: "Drizzle Valley"**
- Color Palette: Gray sky (#A9A9A9), darker clouds (#D3D3D3), wet stone platforms (#708090)
- Enemies: Drizzle Sprites (12), Thunder Sparks (4)
- Platform Layout: Some vertical climbing required, narrower platforms
- Background: Rain effect (falling vertical lines, 1 pixel wide, slow fall)

**Level 3: "Thunder Ridge"**
- Color Palette: Dark purple sky (#483D8B), yellow electric platforms (#FFD700), storm clouds (#2F4F4F)
- Enemies: Thunder Sparks (10), Breeze Wisps (6)
- Platform Layout: Platforms with gaps, requires precise jumps
- Background: Lightning flashes every 120 frames (screen briefly brightens)

**Level 4: "Frost Summit"**
- Color Palette: Pale blue sky (#E0FFFF), icy platforms (#B0E0E6), snowflakes falling
- Enemies: Frost Puffs (8), Drizzle Sprites (4), Thunder Sparks (4)
- Platform Layout: Slippery platforms (player slides 1.2x normal speed), vertical climbing
- Background: Snowfall effect (falling dots, white, slow descent)

**Level 5: "Cumulus Fortress"**
- Color Palette: Dark gray sky (#696969), black storm clouds (#2F4F4F), dark platforms (#3C3C3C)
- Enemies: All enemy types (5 of each)
- Platform Layout: Complex layout with multiple paths, requires all mechanics
- Background: Heavy storm (rain + lightning + wind particles moving)

### UI System

#### HUD Elements (always visible)
- **Health Display:** Top-left corner (x=20, y=20)
  - Shows 6 cloud-shaped health icons (30x25 pixels each)
  - Filled icons = current health, hollow outlines = lost health
  - Player starts with 6 HP maximum
  
- **Boss Health Bar:** Top-center (x=200, y=20) - only visible during boss fights
  - Rectangle (400x30 pixels), border 2px black
  - Inner fill represents boss health (red color #FF0000, depletes right to left)
  - Boss name label above bar (centered, 16px font)

- **Level Name:** Top-right corner (x=600, y=20)
  - Displays current level name (e.g., "Level 1: Gentle Breeze Plains")
  - White text with black stroke, 14px font

#### Title Screen
- **Background:** Animated clouds drifting slowly (2 large SVG clouds moving left at 0.5 pixels/frame, loop)
- **Title:** "Sky Drift Adventures" centered at y=150, large bold text (48px), white with blue shadow
- **Subtitle:** "A Cloud's Journey" at y=200, smaller text (24px), italic
- **Main Menu Options:** Centered at y=300, 32px font
  - "Start Adventure" - begins Level 1
  - "Extra Mode" - grayed out until main game completed, unlocks hard mode
  - "Config" - opens configuration screen
- **Instructions:** Bottom of screen (y=550), small text (12px): "Arrow Keys: Move | Z: Inhale | X: Spit | Space: Jump/Fly"
- **Menu Selection:** Use Up/Down arrows to highlight option (yellow outline), press Space or Z to select

#### Configuration Screen
- **Background:** Simplified version of title screen
- **Options List:** Centered, vertical layout
  - "Starting Lives: [1-9]" - adjustable with Left/Right arrows
  - "Starting Health: [3-6]" - adjustable with Left/Right arrows
  - "Sound: [On/Off]" - toggle with Space (sound is visual only, no actual audio)
- **Back Button:** "Return to Menu" at bottom, returns to title screen

#### Game Over Screen
- **Trigger:** Appears when player health reaches 0
- **Background:** Faded version of last screen (50% opacity black overlay)
- **Message:** "Game Over" centered, large text (56px), white
- **Options:**
  - "Retry Level" - restart current level from beginning
  - "Return to Title" - go back to title screen
- **Selection:** Arrow keys + Space to choose

#### Victory Screen
- **Trigger:** Appears after defeating Level 5 boss
- **Background:** Clearing storm animation (clouds parting, sun rays appearing)
- **Message:** "Sky Realm Restored!" centered, y=200, large text (48px)
- **Ending Text:** Scrolling credits/story conclusion at y=300 (4 sentences, scrolls upward 1 pixel/frame)
- **Unlock Message:** "Extra Mode Unlocked!" appears at y=500 after 300 frames
- **Return:** Automatically returns to title screen after 600 frames (20 seconds)

### Progression System

#### Level Completion
- Touching end portal teleports player to boss arena
- Defeating boss opens exit portal in boss arena
- Touching boss exit portal saves progress and proceeds to next level
- Progress saved to localStorage: `skyDriftProgress: { level: number, extraUnlocked: boolean }`

#### Extra Mode
- Unlocked after completing Level 5
- Changes to gameplay:
  - Player starts with 3 HP instead of 6
  - Enemies have 1.5x normal health (rounded up)
  - Enemies move 1.3x faster
  - Enemy count increases by 50% per level (rounded up, max 30 enemies)
  - Boss health increases by 50%
- Same 5 levels with harder difficulty
- No additional reward for completing Extra Mode (bragging rights only)

#### Death and Respawn
- When player health reaches 0:
  - Play 20-frame puff animation (Nimbus disperses into small cloud particles)
  - Show Game Over screen
  - Retry Level: Player restarts at level beginning with full health
  - Player has infinite retries

### Input Handling

#### Keyboard Controls
- **Arrow Left:** Move left, faces left
- **Arrow Right:** Move right, faces right
- **Arrow Up:** While flying, move up
- **Arrow Down:** While running, trigger slide; while flying, descend
- **Shift:** Hold to run (faster movement)
- **Z Key:** Hold to inhale, release to swallow
- **X Key:** Spit projectile (if holding enemy) or shoot air puff
- **Spacebar:** Jump (on ground), inflate/fly (in air)
- **Escape:** Pause game (show pause menu overlay)

#### Pause Menu (during gameplay)
- Overlay with 60% opacity black background
- Options centered:
  - "Resume" - return to game
  - "Restart Level" - restart current level
  - "Return to Title" - return to title screen
- Use Arrow keys + Space to select

## Technical Implementation

### Architecture

#### Component Structure
The game uses a React component hierarchy with clear separation of concerns:

```
App.jsx (root, 150 lines max)
├── GameStateManager.jsx (game loop, state management, 200 lines max)
│   ├── TitleScreen.jsx (menu UI, 120 lines max)
│   ├── ConfigScreen.jsx (config UI, 100 lines max)
│   ├── GameWorld.jsx (level rendering container, 180 lines max)
│   │   ├── Player.jsx (Nimbus character, 250 lines max)
│   │   ├── EnemyManager.jsx (enemy spawning/management, 150 lines max)
│   │   │   └── Enemy.jsx (individual enemy component, 200 lines max)
│   │   ├── BossManager.jsx (boss battle handler, 180 lines max)
│   │   │   └── Boss.jsx (individual boss component, 300 lines max)
│   │   ├── LevelTerrain.jsx (platforms, background, 150 lines max)
│   │   ├── Camera.jsx (viewport management, 100 lines max)
│   │   └── ProjectileManager.jsx (projectile tracking, 120 lines max)
│   │       └── Projectile.jsx (individual projectile, 80 lines max)
│   ├── HUD.jsx (health, boss health bar, level name, 130 lines max)
│   ├── PauseMenu.jsx (pause overlay, 80 lines max)
│   ├── GameOverScreen.jsx (game over UI, 90 lines max)
│   └── VictoryScreen.jsx (victory UI, 110 lines max)
├── hooks/
│   ├── useGameLoop.js (requestAnimationFrame hook, 60 lines max)
│   ├── useKeyboard.js (keyboard input tracking, 80 lines max)
│   └── useCollision.js (collision detection utilities, 100 lines max)
├── utils/
│   ├── physics.js (gravity, velocity calculations, 80 lines max)
│   ├── collision.js (hitbox intersection checks, 70 lines max)
│   └── levelData.js (level definitions export, 200 lines max)
└── constants/
    ├── gameConstants.js (physics values, speeds, damage, 60 lines max)
    ├── enemyData.js (enemy type definitions, 100 lines max)
    └── bossData.js (boss definitions, 150 lines max)
```

#### State Management Strategy

Primary game state stored in GameStateManager using useState/useReducer:

```javascript
gameState = {
  screen: 'title' | 'config' | 'playing' | 'paused' | 'gameOver' | 'victory',
  currentLevel: 1,
  extraMode: false,
  extraUnlocked: false,
  player: {
    x: number,
    y: number,
    velocityX: number,
    velocityY: number,
    health: number,
    maxHealth: number,
    facing: 'left' | 'right',
    state: 'idle' | 'walking' | 'running' | 'jumping' | 'flying' | 'sliding' | 'inhaling',
    inhaling: boolean,
    inflated: boolean,
    holdingEnemy: null | { type: string, id: number },
    invincible: boolean,
    invincibilityFrames: number
  },
  enemies: [
    {
      id: number,
      type: 'breezeWisp' | 'drizzleSprite' | 'thunderSpark' | 'frostPuff',
      x: number,
      y: number,
      velocityX: number,
      velocityY: number,
      health: number,
      patrolMin: number,
      patrolMax: number,
      animationFrame: 0 | 1,
      animationCounter: number
    }
  ],
  boss: null | {
    type: string,
    health: number,
    maxHealth: number,
    x: number,
    y: number,
    phase: number,
    actionTimer: number,
    invincibilityFrames: number
  },
  projectiles: [
    {
      id: number,
      type: 'spit' | 'airPuff' | 'bossAttack',
      x: number,
      y: number,
      velocityX: number,
      velocityY: number,
      damage: number,
      distanceTraveled: number,
      maxDistance: number,
      owner: 'player' | 'enemy' | 'boss'
    }
  ],
  camera: {
    x: number,
    y: 0
  },
  config: {
    startingLives: number,
    startingHealth: number,
    soundEnabled: boolean
  }
}
```

#### Game Loop Implementation

Use custom `useGameLoop` hook with requestAnimationFrame:

```javascript
// useGameLoop.js
function useGameLoop(callback, running) {
  useEffect(() => {
    if (!running) return;
    
    let frameId;
    let lastTime = performance.now();
    const targetFPS = 30;
    const frameTime = 1000 / targetFPS;
    
    function loop(currentTime) {
      const deltaTime = currentTime - lastTime;
      
      if (deltaTime >= frameTime) {
        callback(deltaTime / 1000); // pass delta in seconds
        lastTime = currentTime - (deltaTime % frameTime);
      }
      
      frameId = requestAnimationFrame(loop);
    }
    
    frameId = requestAnimationFrame(loop);
    return () => cancelAnimationFrame(frameId);
  }, [running, callback]);
}
```

Call order per frame:
1. Process keyboard input (update player input state)
2. Update player physics (gravity, velocity, position)
3. Update player state machine (idle → walking → jumping etc)
4. Update enemies (AI behavior, animation frames)
5. Update boss (behavior patterns, attacks)
6. Update projectiles (movement, lifetime)
7. Check collisions (player-enemy, projectile-enemy, player-projectile)
8. Update camera position (follow player)
9. Render all components (SVG updates via React state changes)

### SVG Graphics System

#### SVG Container
- Root SVG element: 800x600 viewBox, scales to fit screen
- Coordinate system: (0,0) at top-left, +X right, +Y down
- All graphics defined as pure SVG primitives (rect, circle, ellipse, path, polygon)

#### Player (Nimbus) Rendering

Normal State:
```svg
<g id="nimbus" transform="translate(x, y)">
  <!-- Main cloud body -->
  <ellipse cx="0" cy="0" rx="40" ry="30" fill="#F0F8FF" stroke="#000" stroke-width="2" />
  
  <!-- Cloud puffs (3 overlapping circles) -->
  <circle cx="-20" cy="-10" r="18" fill="#F0F8FF" stroke="#000" stroke-width="2" />
  <circle cx="20" cy="-10" r="18" fill="#F0F8FF" stroke="#000" stroke-width="2" />
  <circle cx="0" cy="-20" r="20" fill="#F0F8FF" stroke="#000" stroke-width="2" />
  
  <!-- Eyes (2 circles) -->
  <circle cx="-10" cy="5" r="5" fill="#000" />
  <circle cx="10" cy="5" r="5" fill="#000" />
  
  <!-- Mouth (small arc path) -->
  <path d="M -8,15 Q 0,20 8,15" stroke="#000" stroke-width="2" fill="none" />
</g>
```

Inflated State (scale transform to 1.5x):
```svg
<g transform="translate(x, y) scale(1.5)">
  <!-- Same structure as normal, scaled up -->
</g>
```

Inhaling State (mouth expands):
```svg
<!-- Mouth becomes larger circle during inhale -->
<ellipse cx="0" cy="15" rx="15" ry="10" fill="#000" />

<!-- Inhale particles (3-5 small circles rotating in cone) -->
<circle cx="inhaleConeX" cy="inhaleConeY" r="3" fill="#87CEEB" opacity="0.6" />
<!-- Particles positioned in 60px cone, rotate around mouth -->
```

Sliding State (compressed vertically):
```svg
<g transform="translate(x, y) scale(1, 0.5)">
  <!-- Same structure, flattened -->
</g>
```

#### Enemy Rendering Examples

Breeze Wisp (frame 0 - tight spiral):
```svg
<g transform="translate(x, y)">
  <path d="M 0,-15 Q 10,-10 10,0 Q 10,10 0,15 Q -10,10 -10,0 Q -10,-10 0,-15" 
        fill="none" stroke="#87CEEB" stroke-width="3" />
  <circle cx="0" cy="0" r="4" fill="#87CEEB" />
</g>
```

Breeze Wisp (frame 1 - loose spiral):
```svg
<g transform="translate(x, y)">
  <path d="M 0,-15 Q 15,-8 15,0 Q 15,8 0,15 Q -15,8 -15,0 Q -15,-8 0,-15" 
        fill="none" stroke="#87CEEB" stroke-width="2" />
  <circle cx="0" cy="0" r="4" fill="#87CEEB" />
</g>
```

Drizzle Sprite (frame 0 - round):
```svg
<g transform="translate(x, y)">
  <ellipse cx="0" cy="0" rx="12" ry="17" fill="#4682B4" stroke="#000" stroke-width="1" />
  <circle cx="-4" cy="-4" r="2" fill="#FFF" /> <!-- eye -->
  <circle cx="4" cy="-4" r="2" fill="#FFF" />
</g>
```

Drizzle Sprite (frame 1 - teardrop):
```svg
<g transform="translate(x, y)">
  <path d="M 0,-17 Q 12,0 0,17 Q -12,0 0,-17" fill="#4682B4" stroke="#000" stroke-width="1" />
  <circle cx="-4" cy="-4" r="2" fill="#FFF" />
  <circle cx="4" cy="-4" r="2" fill="#FFF" />
</g>
```

Thunder Spark (frame 0 - jagged):
```svg
<g transform="translate(x, y)">
  <polygon points="0,-20 5,-10 2,0 8,0 0,20 -3,5 -8,5 -3,-10" 
           fill="#FFD700" stroke="#000" stroke-width="1" />
</g>
```

Thunder Spark (frame 1 - straight):
```svg
<g transform="translate(x, y)">
  <polygon points="0,-20 3,-10 2,0 5,0 0,20 -2,5 -5,5 -3,-10" 
           fill="#FFD700" stroke="#000" stroke-width="1" />
</g>
```

Frost Puff (frame 0 - 6-point star):
```svg
<g transform="translate(x, y)">
  <polygon points="0,-17 4,-5 17,0 4,5 0,17 -4,5 -17,0 -4,-5" 
           fill="#E0FFFF" stroke="#00CED1" stroke-width="2" />
</g>
```

Frost Puff (frame 1 - 8-point star):
```svg
<g transform="translate(x, y)">
  <polygon points="0,-17 3,-8 12,-12 5,-3 17,0 5,3 12,12 3,8 0,17 -3,8 -12,12 -5,3 -17,0 -5,-3 -12,-12 -3,-8" 
           fill="#E0FFFF" stroke="#00CED1" stroke-width="2" />
</g>
```

#### Animation System

Implement minimal 2-frame animations:
- Frame duration: 15 game frames (0.5 seconds at 30 FPS)
- Each entity tracks `animationFrame` (0 or 1) and `animationCounter`
- Every game frame, increment counter
- When counter reaches 15, toggle frame (0→1 or 1→0), reset counter to 0
- Render appropriate SVG based on current frame value

#### Platform and Terrain Rendering

Platform Example:
```svg
<rect x="platformX" y="platformY" width="platformWidth" height="20" 
      fill="platformColor" stroke="#000" stroke-width="2" rx="5" />
```

Background layers (parallax not required, but optional):
- Sky: Full viewport rect with gradient fill
- Distant mountains/clouds: Simple shapes, fixed position
- Weather effects: Small animated elements (rain lines, snow dots)

### Collision Detection

#### Hitbox System

Player Hitbox:
- Normal: Rectangle centered on player (x-35, y-25, width: 70, height: 50)
- Sliding: Rectangle (x-35, y-12.5, width: 70, height: 25)
- Inflated: Rectangle (x-52.5, y-37.5, width: 105, height: 75)

Enemy Hitboxes:
- Breeze Wisp: Circle (radius: 15)
- Drizzle Sprite: Rectangle (width: 24, height: 34)
- Thunder Spark: Rectangle (width: 16, height: 40)
- Frost Puff: Circle (radius: 17)

Boss Hitboxes:
- Rectangular, size varies per boss (defined in bossData.js)

Projectile Hitboxes:
- All projectiles: Circle (radius: 8)

Platform Hitboxes:
- Rectangle matching platform visual dimensions

#### Collision Detection Functions

Rectangle-Rectangle (AABB):
```javascript
function rectIntersects(rect1, rect2) {
  return rect1.x < rect2.x + rect2.width &&
         rect1.x + rect1.width > rect2.x &&
         rect1.y < rect2.y + rect2.height &&
         rect1.y + rect1.height > rect2.y;
}
```

Circle-Circle:
```javascript
function circleIntersects(circle1, circle2) {
  const dx = circle1.x - circle2.x;
  const dy = circle1.y - circle2.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  return distance < circle1.radius + circle2.radius;
}
```

Circle-Rectangle (for mixed collisions):
```javascript
function circleRectIntersects(circle, rect) {
  const closestX = Math.max(rect.x, Math.min(circle.x, rect.x + rect.width));
  const closestY = Math.max(rect.y, Math.min(circle.y, rect.y + rect.height));
  const dx = circle.x - closestX;
  const dy = circle.y - closestY;
  return (dx * dx + dy * dy) < (circle.radius * circle.radius);
}
```

Platform Landing Check:
- Player is descending (velocityY > 0)
- Player's bottom edge overlaps platform's top edge (within 5 pixels)
- Player's horizontal position overlaps platform horizontally
- Set player.y to platform.y - playerHeight, set velocityY to 0

#### Inhale Cone Detection

```javascript
function isInInhaleCone(playerX, playerY, playerFacing, enemyX, enemyY) {
  const dx = enemyX - playerX;
  const dy = enemyY - playerY;
  const distance = Math.sqrt(dx * dx + dy * dy);
  
  if (distance > 60) return false; // Outside 60px range
  
  // Check if in front of player
  if (playerFacing === 'right' && dx < 0) return false;
  if (playerFacing === 'left' && dx > 0) return false;
  
  // Check cone angle (45 degrees total, 22.5 each side)
  const angle = Math.atan2(dy, Math.abs(dx));
  return Math.abs(angle) < Math.PI / 8; // 22.5 degrees
}
```

### Physics System

#### Gravity and Velocity

Constants (from gameConstants.js):
```javascript
export const GRAVITY = 0.6;
export const TERMINAL_VELOCITY = 12;
export const JUMP_VELOCITY = -12;
export const INFLATED_GRAVITY = 0.1;
export const WALK_SPEED = 3;
export const RUN_SPEED = 5;
export const FLIGHT_HORIZONTAL_SPEED = 3;
export const FLIGHT_VERTICAL_SPEED = -2;
export const SLIDE_SPEED = 6;
```

Player Update (each frame):
```javascript
// Apply gravity
if (!player.inflated && !player.onGround) {
  player.velocityY += GRAVITY;
  player.velocityY = Math.min(player.velocityY, TERMINAL_VELOCITY);
} else if (player.inflated) {
  player.velocityY += INFLATED_GRAVITY;
}

// Apply horizontal velocity
player.x += player.velocityX;

// Apply vertical velocity
player.y += player.velocityY;

// Check platform collisions
// If landed on platform: player.onGround = true, velocityY = 0
```

Projectile Movement:
```javascript
projectile.x += projectile.velocityX;
projectile.y += projectile.velocityY;
projectile.distanceTraveled += Math.abs(projectile.velocityX) + Math.abs(projectile.velocityY);

if (projectile.distanceTraveled >= projectile.maxDistance) {
  removeProjectile(projectile.id);
}
```

### Data Structures

#### Level Data Definition

```javascript
// levelData.js
export const levels = [
  {
    id: 1,
    name: "Gentle Breeze Plains",
    width: 3200,
    height: 600,
    background: {
      skyColor: "#87CEEB",
      elements: [
        { type: "mountain", x: 500, y: 400, width: 300, height: 200, color: "#B0C4DE" },
        // ... more background elements
      ]
    },
    platforms: [
      { x: 0, y: 550, width: 300, height: 30, color: "#90EE90" },
      { x: 400, y: 500, width: 200, height: 30, color: "#90EE90" },
      { x: 700, y: 450, width: 150, height: 30, color: "#90EE90" },
      // ... 12 more platforms
    ],
    enemies: [
      { type: "breezeWisp", x: 300, y: 200, patrolMin: 200, patrolMax: 400 },
      { type: "drizzleSprite", x: 600, y: 520, patrolMin: 500, patrolMax: 800 },
      // ... 18 more enemies
    ],
    endPortal: { x: 3150, y: 500 },
    bossType: "galeCommander"
  },
  // ... levels 2-5
];
```

#### Enemy Data Definition

```javascript
// enemyData.js
export const enemyTypes = {
  breezeWisp: {
    health: 2,
    speed: 2,
    damage: 1,
    hitbox: { type: "circle", radius: 15 },
    aiType: "flyPatrol"
  },
  drizzleSprite: {
    health: 2,
    speed: 1.5,
    damage: 1,
    hitbox: { type: "rect", width: 24, height: 34 },
    aiType: "walkPatrol"
  },
  // ... more enemy types
};
```

#### Boss Data Definition

```javascript
// bossData.js
export const bossTypes = {
  galeCommander: {
    name: "Gale Commander",
    maxHealth: 20,
    hitbox: { type: "rect", width: 100, height: 100 },
    phases: [
      {
        healthThreshold: 11,
        behavior: "horizontalFloat",
        speed: 3,
        attackPattern: "shootAirPuffs",
        attackInterval: 90,
        projectileCount: 3
      },
      {
        healthThreshold: 0,
        behavior: "horizontalFloat",
        speed: 5,
        attackPattern: "summonEnemies",
        attackInterval: 120,
        summonType: "breezeWisp",
        summonCount: 2
      }
    ]
  },
  // ... more boss types
};
```

### Boss AI Implementation

#### Boss Behavior System

Each boss type has phases defined by health thresholds. Boss component tracks current phase:

```javascript
function updateBoss(boss, player, deltaTime) {
  // Determine current phase
  const currentPhase = boss.phases.find(phase => 
    boss.health >= phase.healthThreshold
  );
  
  // Execute behavior
  switch(currentPhase.behavior) {
    case "horizontalFloat":
      boss.x += currentPhase.speed * boss.direction;
      if (boss.x <= 50 || boss.x >= 750) {
        boss.direction *= -1;
      }
      break;
      
    case "teleport":
      boss.actionTimer--;
      if (boss.actionTimer <= 0) {
        boss.x = Math.random() * 600 + 100;
        boss.y = Math.random() * 400 + 100;
        boss.actionTimer = currentPhase.teleportInterval;
      }
      break;
      
    // ... more behaviors
  }
  
  // Execute attack pattern
  boss.actionTimer--;
  if (boss.actionTimer <= 0) {
    executeAttack(boss, currentPhase, player);
    boss.actionTimer = currentPhase.attackInterval;
  }
}
```

#### Boss Attack Patterns

```javascript
function executeAttack(boss, phase, player) {
  switch(phase.attackPattern) {
    case "shootAirPuffs":
      for (let i = 0; i < phase.projectileCount; i++) {
        createProjectile({
          type: "bossAirPuff",
          x: boss.x + (i - 1) * 30,
          y: boss.y + 50,
          velocityX: 0,
          velocityY: 3,
          damage: 2,
          owner: "boss"
        });
      }
      break;
      
    case "summonEnemies":
      for (let i = 0; i < phase.summonCount; i++) {
        createEnemy({
          type: phase.summonType,
          x: Math.random() * 600 + 100,
          y: 100,
          temporary: true,
          lifetime: 180 // despawns after 180 frames
        });
      }
      break;
      
    case "shootRadial":
      const directions = 8;
      for (let i = 0; i < directions; i++) {
        const angle = (Math.PI * 2 * i) / directions;
        createProjectile({
          type: "lightning",
          x: boss.x,
          y: boss.y,
          velocityX: Math.cos(angle) * 4,
          velocityY: Math.sin(angle) * 4,
          damage: 3,
          owner: "boss"
        });
      }
      break;
      
    // ... more attack patterns
  }
}
```

### LocalStorage Persistence

Save progress after level completion:
```javascript
function saveProgress(level, extraUnlocked) {
  const saveData = {
    completedLevel: level,
    extraModeUnlocked: extraUnlocked,
    timestamp: Date.now()
  };
  localStorage.setItem('skyDriftProgress', JSON.stringify(saveData));
}
```

Load progress on title screen mount:
```javascript
function loadProgress() {
  const saved = localStorage.getItem('skyDriftProgress');
  if (saved) {
    const data = JSON.parse(saved);
    return {
      canContinue: true,
      continueLevel: data.completedLevel + 1,
      extraUnlocked: data.extraModeUnlocked
    };
  }
  return { canContinue: false, continueLevel: 1, extraUnlocked: false };
}
```

### Performance Optimization

#### Component Memoization
- Wrap Enemy, Projectile, Platform components in React.memo()
- Prevent re-renders when props haven't changed
- Use key prop correctly for list items (stable IDs)

#### Update Optimization
- Only check collisions for entities within camera viewport + 100px buffer
- Despawn off-screen enemies when player moves far away (>1000px)
- Limit projectile count to 20 maximum (remove oldest when exceeded)

#### Render Optimization
- Use CSS transform for camera movement instead of re-rendering entire level
- Batch state updates using useReducer for complex state changes
- Avoid inline function creation in render (use useCallback)

## Style Guide

### Visual Design Specifications

#### Color Palette

Primary Colors:
- Sky Blue: #87CEEB (primary background)
- Cloud White: #F0F8FF (player character)
- Platform Green: #90EE90 (Level 1)
- Storm Gray: #A9A9A9 (Level 2)
- Electric Purple: #483D8B (Level 3)
- Ice Blue: #E0FFFF (Level 4)
- Dark Storm: #696969 (Level 5)

Accent Colors:
- Health Red: #FF0000 (damage indication, boss health)
- Warning Yellow: #FFD700 (menu selection, lightning)
- UI White: #FFFFFF (text, outlines)
- UI Black: #000000 (text stroke, borders)

#### Typography

Font Stack: `'Courier New', monospace` (Game Boy aesthetic)

Text Sizes:
- Title: 48px, bold, white with 3px black stroke
- Subtitle: 24px, italic
- Menu Options: 32px, regular
- HUD Text: 16px, regular
- Instructions: 12px, regular
- Boss Names: 20px, bold

Text Rendering:
- All text uses SVG <text> elements
- Stroke for readability: 2px black stroke on white text
- Center alignment for menus and titles
- Left alignment for HUD elements

#### Animation Guidelines

2-Frame Animation Timing:
- 15 frames per animation frame = 0.5 seconds per frame at 30 FPS
- Total loop time: 1 second (frame 0 → frame 1 → frame 0)

Smooth Transitions:
- No easing - all movement is linear
- Immediate state changes (no fade transitions)
- Game Boy-style instant feedback

Special Effects:
- Damage flash: 3 frames white overlay on hit entity
- Invincibility: Flash transparent/opaque every 5 frames
- Death puff: 20-frame expanding circle animation, increasing radius by 2px/frame
- Victory sparkle: 4 random stars appearing around defeated boss, fade over 30 frames

#### UI Layout

Screen Layout (800x600):
- Safe margins: 20px from all edges
- HUD top row: y=20
- Menu content center: y=300
- Bottom instructions: y=550

Element Spacing:
- Menu options: 50px vertical spacing
- HUD icons: 35px horizontal spacing
- Paragraph line height: 1.5x font size

## Testing Scenarios

### Mechanical Testing

#### Movement Tests
1. **Basic Movement**
   - Press Left arrow: Nimbus moves left at 3 pixels/frame, faces left
   - Press Right arrow: Nimbus moves right at 3 pixels/frame, faces right
   - Press Shift+Right: Nimbus moves right at 5 pixels/frame
   - Reach screen edge: Nimbus stops at x=0 or x=800, doesn't go offscreen

2. **Jumping**
   - Press Space on ground: Nimbus jumps with initial velocity -12
   - Nimbus ascends, decelerates, descends under gravity
   - Lands on platform: Stops falling, can jump again
   - Press Space in air (first time): Nimbus inflates, becomes floaty
   - Press Space in air (already inflated): No additional effect

3. **Flying**
   - While inflated, press Up: Nimbus moves up at -2 pixels/frame
   - While inflated, press Down: Nimbus deflates, returns to normal gravity
   - While inflated, press Left/Right: Moves horizontally at 3 pixels/frame
   - Unlimited flight time: Can stay airborne indefinitely

4. **Sliding**
   - While running right, press Down: Nimbus slides right for 30 frames at 6 pixels/frame
   - Slide into enemy: Deals 1 damage
   - Cannot slide while airborne
   - Cannot slide while moving left

#### Combat Tests
1. **Inhale Mechanic**
   - Hold Z facing enemy 50 pixels away: Enemy pulls toward Nimbus at 4 pixels/frame
   - Enemy reaches Nimbus center: Enemy captured, mouth closes
   - Release Z with captured enemy: Enemy swallowed, disappears
   - Press X with captured enemy: Enemy shoots as projectile at 8 pixels/frame
   - Projectile hits another enemy: Deals 2 damage, projectile disappears

2. **Air Puff**
   - Press X without holding enemy: Shoots star-shaped air puff at 6 pixels/frame
   - Air puff travels 80 pixels: Disappears after reaching max range
   - Air puff hits enemy: Deals 1 damage, puff disappears
   - Rapid-fire test: Can shoot air puff every frame (30/second)

3. **Damage System**
   - Nimbus touches enemy: Loses 1 HP, becomes invincible for 15 frames
   - During invincibility: Flash transparent/opaque, cannot take damage
   - After invincibility: Can be damaged again
   - Health reaches 0: Death animation plays, game over screen appears

#### Enemy Behavior Tests
1. **Breeze Wisp**
   - Spawns at x=300, y=200, patrolMin=200, patrolMax=400
   - Moves right at 2 pixels/frame until x=400
   - Reverses direction, moves left to x=200
   - Continues patrol loop indefinitely
   - Takes 2 damage total to defeat (1 air puff + 1 slide, or 1 spit projectile)

2. **Thunder Spark**
   - Stands on platform, jumps every 60 frames
   - Jump velocity -10, follows gravity
   - Lands on platform, waits 60 frames, jumps again
   - Takes 3 damage to defeat (2 air puffs + 1 slide)

3. **Enemy Respawn**
   - Defeat all enemies in level
   - Exit level and return (or die and retry)
   - All enemies respawn at original positions with full health

#### Boss Battle Tests
1. **Gale Commander Phase 1**
   - Boss starts at full 20 HP
   - Floats horizontally at y=100, speed 3 pixels/frame
   - Every 90 frames: Shoots 3 air puffs downward
   - Player hit by air puff: Takes 2 damage
   - Deal 10 damage to boss: Phase 2 triggers

2. **Gale Commander Phase 2**
   - Boss speed increases to 5 pixels/frame
   - Every 120 frames: Summons 2 Breeze Wisps
   - Wisps despawn after 180 frames if not defeated
   - Deal 10 more damage: Boss defeated, exit portal appears

3. **Boss Invincibility**
   - Hit boss with projectile: Boss takes damage, flashes white for 15 frames
   - During flash: Additional hits don't deal damage
   - After flash ends: Boss can be damaged again
   - Rapid-fire test: Maximum damage rate is 1 hit per 15 frames

### UI Testing

1. **Title Screen**
   - Load game: Title screen appears with animated clouds
   - Press Up/Down: Menu selection highlights different options (yellow outline)
   - Press Space on "Start Adventure": Game begins at Level 1
   - Press Space on "Config": Config screen appears

2. **HUD Accuracy**
   - Start game with 6 HP: HUD shows 6 filled cloud icons
   - Take 1 damage: HUD shows 5 filled, 1 hollow icon
   - Boss battle starts: Boss health bar appears with full red fill
   - Deal damage to boss: Health bar depletes proportionally (10 damage = 50% depleted for 20 HP boss)

3. **Game Over Flow**
   - Health reaches 0: Game over screen appears with options
   - Select "Retry Level": Level restarts from beginning, player has full health
   - Select "Return to Title": Returns to title screen, progress not saved

4. **Victory Flow**
   - Defeat Level 5 boss: Victory screen appears
   - Ending text scrolls upward for 10 seconds
   - After 20 seconds: Returns to title screen automatically
   - "Extra Mode" option now enabled (not grayed out)

### Edge Case Testing

1. **Platform Edge Behavior**
   - Walk to platform edge: Nimbus hangs over edge slightly before falling
   - Jump at platform edge: Jump executes normally
   - Land on platform edge (barely touching): Landing succeeds if any overlap

2. **Simultaneous Inputs**
   - Hold Left + Right simultaneously: Nimbus doesn't move (inputs cancel)
   - Press Z + X simultaneously: Inhale takes priority (cannot spit while inhaling)
   - Press Space while holding Down (on ground): Jump executes (Down ignored on ground)

3. **Screen Boundary**
   - Fly to top edge (y=0): Can continue upward but Nimbus clips at y=-10 (minor tolerance)
   - Fall below bottom edge (y=600): Immediate death (not visible to player)
   - Walk to left edge (x=0): Camera stops following, Nimbus stops at x=35 (half body visible)

4. **Projectile Limits**
   - Fire 30 air puffs rapidly: Only 20 projectiles exist simultaneously
   - Oldest projectiles auto-removed when limit exceeded
   - Removed projectiles disappear instantly (no animation)

5. **Enemy Despawn**
   - Move 1200 pixels away from enemy: Enemy despawns (removed from state)
   - Return to enemy spawn area: Enemy respawns at original position
   - Despawned enemies don't count toward active enemy limit

### Progressive Difficulty Testing

1. **Normal Mode Difficulty**
   - Complete Level 1 on first attempt (tutorial difficulty)
   - Complete Level 3 within 3 attempts (moderate challenge)
   - Complete Level 5 within 5 attempts (challenging but fair)

2. **Extra Mode Difficulty**
   - Player starts with 3 HP: Fewer mistakes allowed
   - Level 1 enemies have 3 HP (150% of 2): Require more hits to defeat
   - Boss health 30 HP (150% of 20): Longer battles
   - Enemy speed 1.3x: Faster reaction time required

3. **Boss Difficulty Scaling**
   - Gale Commander (Level 1): Predictable pattern, slow projectiles
   - Volt Tyrant (Level 3): Teleportation requires tracking, multi-directional attacks
   - Lord Cumulus (Level 5): Three phases, all attack types, summons previous bosses' enemies

## Accessibility Requirements

### Keyboard Accessibility
- All game functions accessible via keyboard (no mouse required)
- Clear key bindings displayed on title screen
- No key combinations requiring 3+ simultaneous keys
- Pause menu accessible at any time via Escape

### Visual Clarity
- High contrast between player and background (white on blue)
- Enemy types visually distinct (different shapes and colors)
- Health icons clearly visible (large size, high contrast)
- Damage feedback visible (flash effect on hit)

### Difficulty Options
- Config screen allows reducing difficulty (extra starting health)
- Infinite retries available (no game over limit)
- Flight mechanics allow skipping difficult platforming sections
- Boss patterns learnable through repetition (deterministic behavior)

### Text Readability
- All text has black stroke for contrast on any background
- Font size minimum 12px (readable at 800x600 resolution)
- Important info (health, boss health) always visible
- Instructions provided on title screen

## Performance Goals

### Frame Rate
- Target: Stable 30 FPS on modern browsers
- Minimum: 25 FPS acceptable during busy boss fights
- Measurement: No dropped frames during normal gameplay (single player, 15 enemies on screen)

### Load Times
- Initial page load: < 2 seconds
- Level transition: < 0.5 seconds
- Boss arena load: < 0.3 seconds
- LocalStorage read/write: < 0.1 seconds

### Memory Usage
- Maximum DOM elements: ~200 (all SVG elements combined)
- Component instances: ~50 active React components during gameplay
- State object size: < 50KB
- No memory leaks: Memory stable over 30-minute play session

### Responsiveness
- Input lag: < 2 frames (67ms) from key press to visual response
- Collision detection: Complete within single frame (33ms at 30 FPS)
- State updates: < 20ms per frame for all calculations

## Extended Features (Optional Enhancements)

### Future Enhancement Ideas
1. **Sound System**
   - Web Audio API for background music and sound effects
   - Separate volume controls for music and SFX
   - Mute/unmute toggle in config

2. **Additional Levels**
   - Levels 6-10 with new enemy types
   - New boss mechanics (multi-phase transformations)
   - Secret bonus levels

3. **Collectibles**
   - Star pickups for scoring system
   - Health restore items
   - 1-up extra life items

4. **Checkpoints**
   - Mid-level save points
   - Respawn at checkpoint instead of level start

5. **Leaderboard**
   - Time-based scoring
   - Online leaderboard via API
   - Personal best tracking

6. **Mobile Support**
   - Touch controls (virtual D-pad and buttons)
   - Responsive layout for mobile screens
   - Gyroscope controls for tilt-based movement

7. **Accessibility Enhancements**
   - Colorblind modes (alternative color palettes)
   - Reduced motion option (disable particle effects)
   - Text-to-speech for UI elements
   - Fully customizable key bindings

These enhancements are not part of the core specification and should only be implemented after the base game is complete and functional.

---

## Implementation Notes

This README provides complete specifications for implementing Sky Drift Adventures. Developers should:
1. Read entire README before beginning implementation
2. Follow component architecture exactly to maintain file length limits
3. Implement features in order: movement → combat → enemies → bosses → UI
4. Test each system thoroughly using provided test scenarios
5. Ensure all edge cases are handled as specified
6. Meet performance goals before considering optional enhancements

The game should be fully functional and playable when implemented according to these specifications, providing a nostalgic platforming experience reminiscent of classic Game Boy titles while offering modern React development practices.
