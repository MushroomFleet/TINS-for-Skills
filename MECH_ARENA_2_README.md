# MECH ARENA 2: Architect's Protocol

A spiritual successor to Armored Core: Formula Front — a mech customization and automated battle simulation game where players design unmanned mechs for deterministic combat. Featuring stigmergic multiplayer, a full level editor, and offline-first architecture.

## Description

MECH ARENA 2: Architect's Protocol is a React-based indie game where players assume the role of an "Architect" — designing, customizing, and configuring AI-controlled mechs (u-Mechs) that battle autonomously. Players never directly control their mechs during combat; instead, victory depends on engineering excellence: optimal part selection, energy management, brain chip selection, and behavior slider tuning.

The game runs **entirely offline** with full functionality. An optional online component provides asynchronous stigmergic multiplayer — players leave `.MECHA` data signals on a shared database, which other players encounter as opponents in tournaments. Battle results are deterministic and calculated client-side, with immersive top-down 2D arena replays delivered via in-game mail.

**Core Fantasy:** You are the engineer behind the machine, not the pilot inside it.

**Design Philosophy:** Multiplayer without multiplayer. Stigmergic signals replace real-time connections.

---

## Functionality

### 1. Core Game Loop

```
[Hangar] → [Build Mech] → [Configure AI] → [Test Battle] → [Enter Tournament] → [Watch Replay] → [Iterate]
```

#### 1.1 Hangar (Main Hub)
- Central navigation screen displaying player's mech roster
- Quick-access to Builder, AI Lab, Level Editor, Tournament Registry, Mail, and Profile
- Shows current rank, tournament notifications, and mech status
- Maximum 10 mech slots per player (expandable via progression)
- **Creative Mode Toggle:** Unlocks all parts and progression for unrestricted building

#### 1.2 Mech Builder
The assembly interface where players construct mechs from parts across 12 categories:

| Category | Slot Count | Description |
|----------|------------|-------------|
| Head | 1 | Sensors, lock speed, processing capacity |
| Core | 1 | Central chassis, defines mech type (Standard/Boost/Drone) |
| Arms | 2 | Accuracy, manipulation, weapon stability |
| Legs | 1 | Mobility type (Biped/Reverse/Quad/Tank/Hover) |
| Generator | 1 | Energy output and capacity |
| Radiator | 1 | Heat dissipation rate |
| Booster | 1 | Thrust power, energy consumption |
| Right Weapon | 1 | Primary weapon slot |
| Left Weapon | 1 | Secondary weapon slot |
| Back Unit L | 1 | Support/heavy weapons |
| Back Unit R | 1 | Support/heavy weapons |
| Internal | 3 | Optional modules (shields, ECM, repair) |
| Brain Chip | 1 | **REQUIRED** - Animal-themed behavior augmentation |

**Build Constraints:**
- Total weight must not exceed leg capacity
- Energy drain must not exceed generator output × 1.2
- Heat generation must not exceed radiator capacity × 1.5
- Brain chip is mandatory (RAT chip is default for new mechs)

**Part Statistics (per part type):**
```
Common Stats: weight, cost, energy_drain
Head: sensor_range, lock_speed, ecm_resistance
Core: armor, stability, special_type (none|boost|drone)
Arms: accuracy_bonus, recoil_control, melee_damage
Legs: weight_capacity, move_speed, turn_speed, jump_height, mobility_type
Generator: output, capacity, recharge_rate
Radiator: cooling_rate, emergency_coolant_amount
Booster: thrust, consumption, heat_per_boost
Weapons: damage, range, fire_rate, ammo, projectile_speed, stagger_value, weapon_type (bullet|laser|missile)
Internal: varies by module type
Brain Chip: animal_type, ability_name, effect_modifier, level
```

#### 1.3 AI Configuration (Behavior System)

**Behavior Sliders (5-Point System)**
All mechs are configured using five core behavior sliders (0-100 each):

| Slider | Description | Low Value | High Value |
|--------|-------------|-----------|------------|
| Aggression | Distance preference, attack frequency | Maintains range, calculated shots | Closes distance, constant pressure |
| Caution | Cover usage, retreat threshold | Ignores cover, fights to death | Uses cover extensively, retreats early |
| Mobility | Movement frequency, boost usage | Static positioning, conserves boost | Constant movement, liberal boost usage |
| Focus | Target commitment vs. switching | Opportunistic targeting | Commits to single target until destroyed |
| Energy Conservation | Resource management priority | Ignores energy limits, maximum output | Careful energy budgeting, sustained fire |

**Brain Chips (Animal-Themed Behavior Augmentation)**
Brain chips provide unique behavioral abilities that augment slider-based AI. Each chip has a thematic animal and corresponding ability:

| Chip Name | Animal | Ability | Effect |
|-----------|--------|---------|--------|
| RAT | Rat | Scavenger Instinct | Default chip. Balanced baseline behavior, slight accuracy bonus when low HP |
| BUNNY | Rabbit | Leap Forward | Periodic forward jump in movement pattern, dodges incoming fire |
| TURTLE | Turtle | Shell Protocol | Enhanced defensive behavior, reduces incoming damage while stationary |
| HAWK | Hawk | Precision Strike | Improved lock-on speed, bonus damage to first hit on new target |
| WOLF | Wolf | Pack Tactics | Increased aggression when enemy is damaged below 50% |
| SNAKE | Snake | Ambush Pattern | Remains still until enemy in optimal range, first strike bonus |
| BULL | Bull | Charge Protocol | Aggressive close-range rushes, increased stagger damage |
| SPIDER | Spider | Web Positioning | Prefers high ground and corners, area denial behavior |
| SHARK | Shark | Blood Frenzy | Damage increases as enemy HP decreases |
| OWL | Owl | Night Watch | Enhanced sensor range, detects enemies using cover |
| BEAR | Bear | Endurance Mode | HP regeneration when not taking damage for 3 seconds |
| MANTIS | Mantis | Counter Strike | Increased damage when hit during previous tick |
| CHEETAH | Cheetah | Sprint Burst | Periodic extreme speed boost, high energy cost |
| RHINO | Rhino | Unstoppable Force | Reduced stagger, increased collision damage |
| SCORPION | Scorpion | Poison Protocol | Attacks apply damage-over-time effect |
| PHOENIX | Phoenix | Second Wind | Once per battle: survives destruction with 10% HP |

**Chip Levels:**
Each brain chip has 5 levels, with increasing effect magnitude:
- Level 1: Base effect (unlocked by default or early progression)
- Level 2: 20% stronger effect
- Level 3: 40% stronger effect
- Level 4: 60% stronger effect
- Level 5: 80% stronger effect + visual flair

**Important:** A brain chip is always required. New mechs automatically receive RAT Level 1.

#### 1.4 Battle Simulation
Deterministic combat resolution with visual top-down 2D playback:

**Battle Parameters:**
- Arena size: Small (50m²), Medium (100m²), Large (200m²)
- Arena type: Open, Urban, Industrial, Canyon, Custom (player-created)
- Time limit: 180 seconds default
- Win conditions: Destroy opponent or highest HP% at timeout

**Simulation Tick Rate:** 60 ticks/second (16.67ms per tick)

**Combat Calculations per Tick:**
```
1. AI Decision Phase
   - Read behavior slider values
   - Apply brain chip ability modifiers
   - Calculate action based on current state
   
2. Movement Phase
   - Apply movement vectors
   - Check collision with terrain (buildings slow movement, don't block)
   - Check water tiles (heavy movement penalty)
   - Check ice tiles (low friction, momentum-based)
   - Apply boost if active (drain energy, add heat)
   
3. Combat Phase
   - Process weapon fire commands
   - Check line-of-sight:
     • Bullets/Lasers: BLOCKED by buildings
     • Missiles: PASS THROUGH buildings unimpeded
   - Calculate hit probability: base_accuracy × distance_modifier × movement_modifier
   - Apply damage if hit: base_damage × armor_modifier × critical_modifier
   - Apply stagger if threshold exceeded
   - Apply brain chip combat effects
   
4. Resource Phase
   - Regenerate energy (generator.output / 60)
   - Dissipate heat (radiator.cooling_rate / 60)
   - Check overheat (heat > radiator.capacity × 1.5 → forced cooldown)
   - Check energy shortage (energy < 0 → disable boosters/energy weapons)
   
5. Status Phase
   - Update status effects (stagger recovery, cooldowns, DOT effects)
   - Check destruction (hp <= 0)
   - Check brain chip triggered abilities (Phoenix revival, etc.)
```

**Arena Terrain Types:**
| Terrain | Movement Effect | Bullet/Laser | Missile |
|---------|-----------------|--------------|---------|
| Open Ground | Normal speed | Pass | Pass |
| Building | 50% speed reduction | **Blocked** | Pass |
| Water | 70% speed reduction | Pass | Pass |
| Ice | Normal speed, low friction | Pass | Pass |

#### 1.5 .MECHA File Format
The universal mech data format, stored as JSON:

```json
{
  "format_version": "2.0",
  "mecha_id": "uuid-v4",
  "name": "Nightshade",
  "architect": "PlayerName",
  "created": "ISO8601",
  "modified": "ISO8601",
  
  "parts": {
    "head": "part_id",
    "core": "part_id",
    "leftArm": "part_id",
    "rightArm": "part_id",
    "legs": "part_id",
    "generator": "part_id",
    "radiator": "part_id",
    "booster": "part_id",
    "rightWeapon": "part_id",
    "leftWeapon": "part_id",
    "backLeft": "part_id",
    "backRight": "part_id",
    "internal": ["part_id", "part_id", "part_id"]
  },
  
  "brain_chip": {
    "type": "TURTLE",
    "level": 3
  },
  
  "behavior": {
    "aggression": 45,
    "caution": 70,
    "mobility": 30,
    "focus": 85,
    "energy_conservation": 50
  },
  
  "cosmetics": {
    "primaryColor": "#2a4858",
    "secondaryColor": "#1a2f3a",
    "accentColor": "#ff6b35",
    "decals": []
  },
  
  "stats_snapshot": {
    "total_weight": 4500,
    "weight_capacity": 5000,
    "energy_output": 800,
    "energy_drain": 650,
    "heat_capacity": 1200,
    "heat_generation": 900,
    "armor_rating": 450,
    "mobility_rating": 65
  },
  
  "tournament_data": {
    "rank": 1650,
    "wins": 47,
    "losses": 23,
    "submitted": "ISO8601"
  }
}
```

**Storage:**
- Local: IndexedDB (primary storage for player's mechs)
- Export: JSON file download for community sharing
- Server: Database mirrors .MECHA JSON structure exactly

#### 1.6 Level Editor
Full-featured arena creation tool:

**Editor Features:**
- **Spawn Placement:** Set spawn points for both mechs (Player/Opponent)
- **Building Placement:** Add, move, scale, rotate building objects
- **Terrain Painting:** Place water tiles, ice patches
- **Object Library:** Pre-made SVG assets for buildings, obstacles, decorations
- **Grid Snapping:** Optional snap-to-grid for precise placement
- **Test Mode:** Instantly test arena with current mech vs. training dummy

**Arena Data Format:**
```json
{
  "format_version": "1.0",
  "arena_id": "uuid-v4",
  "name": "Urban Showdown",
  "author": "PlayerName",
  "size": "medium",
  "dimensions": { "width": 100, "height": 100 },
  
  "spawns": {
    "player": { "x": 10, "y": 50, "rotation": 0 },
    "opponent": { "x": 90, "y": 50, "rotation": 180 }
  },
  
  "objects": [
    {
      "type": "building",
      "asset_id": "building_warehouse_01",
      "position": { "x": 30, "y": 40 },
      "scale": { "x": 1.0, "y": 1.0 },
      "rotation": 0
    },
    {
      "type": "terrain",
      "terrain_type": "water",
      "bounds": { "x": 45, "y": 20, "width": 10, "height": 30 }
    },
    {
      "type": "terrain",
      "terrain_type": "ice",
      "bounds": { "x": 70, "y": 60, "width": 15, "height": 15 }
    }
  ]
}
```

**SVG Asset Requirements:**
- All arena graphics are detailed SVG assets
- Assets designed at high resolution for quality scaling
- Buildings have defined collision bounds separate from visual bounds
- Color theming via CSS custom properties

#### 1.7 Tournament System (Stigmergic Multiplayer)

**Philosophy:** Players leave .MECHA data as signals on a shared database. Other players encounter these as opponents. No real-time connection required.

**How It Works:**
1. Player completes local build and testing
2. Player enters tournament (triggers server fetch)
3. Server returns random selection of .MECHA data (clustered by rank)
4. Client caches opponent roster locally
5. All battles simulated client-side using deterministic engine
6. Results calculated instantly with full telemetry for replays
7. Roster remains cached until player wins tournament
8. Winning triggers new roster fetch from server

**Tournament Types:**
- **Training:** Uses bundled fallback .MECHA data (always available offline)
- **Quick Match:** Immediate 1v1 against cached opponent
- **Local Tournament:** 8-mech bracket using cached roster
- **Ranked Season:** Continuous ladder (when online features enabled)

**Server Interaction (Minimal):**
```
READ:  GET /mecha/random?rank=1500&count=8  → Returns 8 .MECHA files near rank
WRITE: POST /mecha/submit                    → Mirrors .MECHA to database (PLANNED)
```

**Current Limitation:** .MECHA submission to server is not implemented in this version. Workaround: Developers curate a GitHub repository of community .MECHA files. Server admins pull from this repository to populate the database.

#### 1.8 Battle Replay System
Full telemetry playback from deterministic simulation:

**Replay Features:**
- Top-down 2D arena view with detailed SVG mech renders
- Real-time position, rotation, and action visualization
- Health/energy/heat bars per mech
- Event markers (hits, critical hits, staggers, ability triggers)
- Timeline scrubber for seeking to any tick
- Speed controls: 0.25x, 0.5x, 1x, 2x, 4x
- Frame-by-frame advance/rewind

**Battle Log Format:**
```json
{
  "battle_id": "uuid",
  "seed": 123456,
  "arena": { "arena_id": "uuid", "name": "Urban Showdown" },
  "mechs": {
    "player": { "...full .MECHA snapshot..." },
    "opponent": { "...full .MECHA snapshot..." }
  },
  "ticks": [
    {
      "tick": 0,
      "player": {
        "position": [10.000, 50.000],
        "rotation": 0.0,
        "hp": 1000,
        "energy": 500,
        "heat": 0,
        "action": "move_forward",
        "brain_chip_active": false
      },
      "opponent": {
        "position": [90.000, 50.000],
        "rotation": 180.0,
        "hp": 1000,
        "energy": 500,
        "heat": 0,
        "action": "strafe_right",
        "brain_chip_active": false
      },
      "events": []
    }
  ],
  "result": {
    "winner": "player",
    "reason": "destruction",
    "duration_ticks": 4523,
    "duration_seconds": 75.38
  }
}
```

#### 1.9 Mail System
In-game notification center with embedded replay access:

**Mail Types:**
- **Battle Results:** Win/loss, opponent name, rating change, embedded replay
- **Tournament Updates:** Round results, bracket position, final standings
- **System Announcements:** Patch notes, events
- **Achievement Unlocks:** New parts, chips, cosmetics earned

**Mail Data Structure:**
```json
{
  "id": "uuid",
  "type": "battle_result",
  "timestamp": "ISO8601",
  "read": false,
  "title": "Victory vs. SteelHunter",
  "body": "Your mech 'Nightshade' defeated 'Iron Wolf' in 2:34",
  "data": {
    "battle_log": { "...complete battle log for replay..." },
    "rating_change": 15
  }
}
```

#### 1.10 Profile & Progression

**Two Progression Modes:**

**Standard Mode:**
Unlock parts and brain chips through gameplay.

| Level | Unlock |
|-------|--------|
| 1 | Base parts (40), RAT brain chip |
| 3 | BUNNY, TURTLE chips |
| 5 | Intermediate parts (30), HAWK chip |
| 8 | WOLF, SNAKE chips |
| 10 | Advanced parts (25), BULL chip |
| 12 | SPIDER, SHARK chips |
| 15 | Expert parts (20), OWL chip |
| 18 | BEAR, MANTIS chips |
| 20 | Elite parts (15), CHEETAH chip |
| 23 | RHINO, SCORPION chips |
| 25 | Legendary parts (10), PHOENIX chip |
| 30 | All chips at Level 5 |

**Creative Mode:**
Instantly unlocks all parts, all brain chips at all levels, and all cosmetics. Designed for:
- Architects who want to experiment freely
- Community builders sharing designs
- Testing and balancing during development

**No penalties or restrictions for Creative Mode users.** Full tournament participation allowed. Design experimentation is encouraged.

**XP Sources:**
- Complete battle: 50 XP
- Win battle: +100 XP bonus
- Daily first win: +200 XP bonus
- Tournament participation: 100 XP per round
- Achievement completion: varies

---

## Technical Implementation

### Architecture Overview

```
src/
├── components/           # React UI components
│   ├── common/          # Shared UI elements (buttons, modals, sliders)
│   ├── hangar/          # Main hub, mech roster
│   ├── builder/         # Part selection, mech assembly
│   ├── ai-lab/          # Behavior sliders, brain chip selection
│   ├── level-editor/    # Arena creation tools
│   ├── battle/          # Battle viewer, replay controls
│   ├── tournament/      # Tournament browser, queue
│   └── mail/            # Inbox, message viewer
├── engine/
│   ├── simulation/      # Deterministic battle engine
│   ├── ai/              # Behavior system, brain chip logic
│   └── replay/          # Battle log playback
├── data/
│   ├── parts/           # Part definitions (JSON)
│   ├── brain-chips/     # Brain chip definitions (JSON)
│   ├── arenas/          # Default arena definitions
│   ├── fallback-mechs/  # Bundled .MECHA files for offline play
│   └── assets/          # SVG graphics, sounds
├── services/
│   ├── storage/         # IndexedDB wrapper
│   ├── mecha-io/        # .MECHA import/export
│   ├── arena-io/        # Arena import/export
│   └── server/          # Optional server communication
├── hooks/               # React custom hooks
├── utils/               # Helpers, math, RNG
└── App.jsx              # Root component
```

### State Management

```javascript
// Core application state
const AppState = {
  player: {
    id: "uuid",
    name: "Architect",
    level: 1,
    xp: 0,
    rank: 1500,
    creativeMode: false,
    unlocks: {
      parts: ["part_id", ...],
      brainChips: { "RAT": 1 }  // chip_type: max_level_unlocked
    }
  },
  mechs: [
    { /* .MECHA object */ }
  ],
  customArenas: [
    { /* Arena object */ }
  ],
  cachedOpponents: [
    { /* .MECHA objects from server */ }
  ],
  mail: [
    { /* Mail objects with embedded battle logs */ }
  ],
  settings: {
    sfxVolume: 0.8,
    musicVolume: 0.5,
    reducedMotion: false,
    autoSave: true
  }
};
```

### .MECHA Storage and Sync

**Local Storage (IndexedDB):**
```javascript
// IndexedDB schema
{
  stores: {
    mechs: {
      keyPath: 'mecha_id',
      indexes: ['name', 'modified', 'rank']
    },
    arenas: {
      keyPath: 'arena_id',
      indexes: ['name', 'author']
    },
    opponents: {
      keyPath: 'mecha_id',
      indexes: ['rank', 'cached_at']
    },
    battles: {
      keyPath: 'battle_id',
      indexes: ['timestamp']
    },
    mail: {
      keyPath: 'id',
      indexes: ['timestamp', 'read', 'type']
    },
    settings: {
      keyPath: 'key'
    }
  }
}
```

**Import/Export:**
```javascript
// Export .MECHA to JSON file
function exportMecha(mecha) {
  const json = JSON.stringify(mecha, null, 2);
  const blob = new Blob([json], { type: 'application/json' });
  downloadBlob(blob, `${mecha.name}.mecha.json`);
}

// Import .MECHA from JSON file
async function importMecha(file) {
  const json = await file.text();
  const mecha = JSON.parse(json);
  validateMechaSchema(mecha);  // Throws if invalid
  mecha.mecha_id = generateUUID();  // New ID on import
  await saveMechaToIndexedDB(mecha);
  return mecha;
}
```

### Deterministic Battle Engine

**Core Requirements:**
- Fixed-point arithmetic for positions (1/1000 unit precision)
- Seeded PRNG (Mulberry32) for all random events
- Sorted entity processing order (by ID)
- Frame-independent physics (fixed timestep)
- Brain chip effects applied deterministically

**Engine Interface:**
```javascript
class BattleEngine {
  constructor(playerMecha, opponentMecha, arena, seed) {
    this.state = initializeBattleState(playerMecha, opponentMecha, arena);
    this.rng = createSeededRNG(seed);
    this.log = { ticks: [], events: [] };
    this.brainChips = {
      player: loadBrainChip(playerMecha.brain_chip),
      opponent: loadBrainChip(opponentMecha.brain_chip)
    };
  }
  
  tick() {
    this.processAI();
    this.processMovement();
    this.processLineOfSight();
    this.processCombat();
    this.processResources();
    this.processBrainChipEffects();
    this.processStatus();
    this.log.ticks.push(this.captureState());
    return !this.checkEndCondition();
  }
  
  runToCompletion() {
    while (this.tick() && this.state.tick < MAX_TICKS) {}
    return this.generateBattleLog();
  }
}
```

**Movement and Terrain:**
```javascript
function calculateMovement(mech, terrain, delta) {
  let speedMultiplier = 1.0;
  
  // Check terrain at mech position
  const tile = terrain.getTileAt(mech.position);
  switch (tile.type) {
    case 'building':
      speedMultiplier = 0.5;  // 50% speed
      break;
    case 'water':
      speedMultiplier = 0.3;  // 30% speed
      break;
    case 'ice':
      // Normal speed but apply momentum/friction model
      mech.velocity = applyIceFriction(mech.velocity, mech.targetDirection);
      break;
  }
  
  return mech.baseSpeed * speedMultiplier * delta;
}
```

**Line of Sight for Weapons:**
```javascript
function checkLineOfSight(from, to, arena, weaponType) {
  if (weaponType === 'missile') {
    // Missiles ignore buildings
    return true;
  }
  
  // Bullets and lasers blocked by buildings
  const ray = castRay(from, to);
  for (const obj of arena.objects) {
    if (obj.type === 'building' && rayIntersects(ray, obj.bounds)) {
      return false;
    }
  }
  return true;
}
```

### Brain Chip System

```javascript
// brain-chips/definitions.json
{
  "RAT": {
    "name": "RAT",
    "animal": "Rat",
    "ability": "Scavenger Instinct",
    "description": "Balanced baseline. Slight accuracy bonus when HP below 30%.",
    "levels": {
      "1": { "accuracy_bonus": 0.05, "hp_threshold": 0.30 },
      "2": { "accuracy_bonus": 0.07, "hp_threshold": 0.35 },
      "3": { "accuracy_bonus": 0.09, "hp_threshold": 0.40 },
      "4": { "accuracy_bonus": 0.11, "hp_threshold": 0.45 },
      "5": { "accuracy_bonus": 0.15, "hp_threshold": 0.50 }
    }
  },
  "BUNNY": {
    "name": "BUNNY",
    "animal": "Rabbit",
    "ability": "Leap Forward",
    "description": "Periodic forward jump. Grants brief invulnerability.",
    "levels": {
      "1": { "cooldown_ticks": 300, "leap_distance": 5, "invuln_ticks": 6 },
      "2": { "cooldown_ticks": 270, "leap_distance": 6, "invuln_ticks": 8 },
      "3": { "cooldown_ticks": 240, "leap_distance": 7, "invuln_ticks": 10 },
      "4": { "cooldown_ticks": 210, "leap_distance": 8, "invuln_ticks": 12 },
      "5": { "cooldown_ticks": 180, "leap_distance": 10, "invuln_ticks": 15 }
    }
  },
  "TURTLE": {
    "name": "TURTLE",
    "animal": "Turtle",
    "ability": "Shell Protocol",
    "description": "Reduced damage while stationary. Ideal for tank builds.",
    "levels": {
      "1": { "damage_reduction": 0.15, "stationary_ticks_required": 30 },
      "2": { "damage_reduction": 0.20, "stationary_ticks_required": 25 },
      "3": { "damage_reduction": 0.25, "stationary_ticks_required": 20 },
      "4": { "damage_reduction": 0.30, "stationary_ticks_required": 15 },
      "5": { "damage_reduction": 0.40, "stationary_ticks_required": 10 }
    }
  },
  "PHOENIX": {
    "name": "PHOENIX",
    "animal": "Phoenix",
    "ability": "Second Wind",
    "description": "Once per battle: survive destruction with HP restored.",
    "levels": {
      "1": { "revive_hp_percent": 0.10 },
      "2": { "revive_hp_percent": 0.15 },
      "3": { "revive_hp_percent": 0.20 },
      "4": { "revive_hp_percent": 0.25 },
      "5": { "revive_hp_percent": 0.35 }
    }
  }
  // ... remaining 12 chips follow same pattern
}
```

### Level Editor Implementation

```jsx
const LevelEditor = () => {
  const [arena, setArena] = useState(createEmptyArena('medium'));
  const [selectedTool, setSelectedTool] = useState('select');
  const [selectedObject, setSelectedObject] = useState(null);
  
  const tools = ['select', 'spawn', 'building', 'water', 'ice', 'delete'];
  
  const handleCanvasClick = (position) => {
    switch (selectedTool) {
      case 'spawn':
        setArena(placeSpawn(arena, position, spawnType));
        break;
      case 'building':
        setArena(placeBuilding(arena, position, currentAsset));
        break;
      case 'water':
      case 'ice':
        setArena(paintTerrain(arena, position, selectedTool));
        break;
      case 'delete':
        setArena(removeObjectAt(arena, position));
        break;
    }
  };
  
  const handleTransform = (objectId, transform) => {
    // transform: { position, scale, rotation }
    setArena(updateObject(arena, objectId, transform));
  };
  
  return (
    <div className="level-editor">
      <Toolbar tools={tools} selected={selectedTool} onSelect={setSelectedTool} />
      <AssetLibrary onSelect={setCurrentAsset} />
      <EditorCanvas
        arena={arena}
        selectedObject={selectedObject}
        onSelect={setSelectedObject}
        onClick={handleCanvasClick}
        onTransform={handleTransform}
      />
      <PropertyPanel object={selectedObject} onChange={handleTransform} />
      <EditorActions
        onTest={() => testArena(arena)}
        onExport={() => exportArena(arena)}
        onImport={(file) => importArena(file).then(setArena)}
        onSave={() => saveArena(arena)}
      />
    </div>
  );
};
```

### Server API (Optional Feature)

**Minimal REST API:**
```
GET  /mecha/random?rank=1500&count=8&variance=200
     → Returns array of .MECHA JSON objects

POST /mecha/submit  (PLANNED - not implemented)
     → Receives .MECHA JSON, stores in database
     
GET  /mecha/leaderboard?limit=100
     → Returns top mechs by win rate (future feature)
```

**Server Architecture:**
- Database mirrors .MECHA JSON structure exactly
- No user accounts required (anonymous stigmergic play)
- Rank clustering for matchmaking (±200 by default)
- GitHub repo integration for content curation

**Current Workflow (Without Direct Submission):**
1. Players export .MECHA files manually
2. Community shares via Discord/forums
3. Developers curate submissions to GitHub repo
4. Server admins trigger pull from GitHub
5. Database populated with curated mechs

---

## User Interface Specifications

### Screen Flow

```
                    ┌─────────────┐
                    │   Splash    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
              ┌─────│   Hangar    │─────┐
              │     └──────┬──────┘     │
              │            │            │
    ┌─────────▼───┐  ┌─────▼─────┐  ┌───▼─────────┐
    │   Builder   │  │  AI Lab   │  │  Tournament │
    └──────┬──────┘  └─────┬─────┘  └──────┬──────┘
           │               │               │
           └───────────────┼───────────────┘
                           │
                    ┌──────▼──────┐
                    │   Battle    │
                    │   Viewer    │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼──────┐  ┌──▼──┐  ┌──────▼──────┐
       │Level Editor │  │Mail │  │  Profile    │
       └─────────────┘  └─────┘  └─────────────┘
```

### Hangar Screen
- **Layout:** 3-column grid on desktop, scrollable list on mobile
- **Mech Cards:** Name, preview thumbnail, brain chip icon, quick stats
- **Actions:** New Mech, Edit, Duplicate, Delete, Quick Test, Export
- **Navigation:** Sidebar with Profile, Mail (badge), Tournament, Level Editor, Settings
- **Mode Toggle:** Standard / Creative Mode switch

### Builder Screen
- **Layout:** 3-panel (Part List | Mech Preview | Stats Panel)
- **Part List:** Filter by category, search, shows owned vs. locked (grayed in Standard Mode)
- **Mech Preview:** Live-updating SVG composite with rotation/zoom
- **Stats Panel:** Real-time validation bars (weight, energy, heat)
- **Brain Chip Slot:** Prominent slot showing current chip with swap button

### AI Lab Screen
- **Sliders Panel:** 5 horizontal sliders with numeric readouts
- **Slider Labels:** Clear descriptions of low/high behavior
- **Brain Chip Grid:** All 16 chips displayed, locked ones grayed
- **Chip Preview:** Hover shows description, levels, current effects
- **Test Button:** Quick battle with training opponent using current config

### Level Editor Screen
- **Canvas:** Large SVG arena view with grid overlay
- **Toolbar:** Tool selection (Select, Spawn, Building, Water, Ice, Delete)
- **Asset Library:** Scrollable grid of SVG building/obstacle assets
- **Property Panel:** Position, scale, rotation inputs for selected object
- **Actions Bar:** Test, Save, Export JSON, Import JSON

### Battle Viewer Screen
- **Arena View:** Top-down 2D SVG arena with mech sprites
- **Mech Overlays:** HP bars, energy bars, brain chip activation indicators
- **Event Log:** Scrolling feed of combat events
- **Timeline:** Full-width scrubber with event markers
- **Controls:** Play, Pause, Step, Speed selector (0.25x to 4x)

### Tournament Screen
- **Mode Tabs:** Training | Quick Match | Tournament | Ranked
- **Roster Display:** Cached opponents with preview cards
- **Queue Status:** Progress indicator when fetching new roster
- **Bracket View:** Visual tournament bracket with results

### Mail Screen
- **Inbox List:** Type icons, read/unread state, timestamps
- **Message View:** Full content with embedded replay button
- **Replay Integration:** Watch Battle button opens Battle Viewer

---

## Accessibility Requirements

- Full keyboard navigation for all interactive elements
- ARIA labels for SVG graphics and custom controls
- Color-blind friendly palette (no red/green-only indicators)
- Minimum 4.5:1 contrast ratio for all text
- Screen reader announcements for battle events
- Reduced motion option (disables animations, instant transitions)
- Scalable UI (supports 100%-200% browser zoom)
- Brain chip icons include text labels, not just animal icons

---

## Performance Goals

- Initial load: < 3 seconds on 3G connection
- Battle simulation: 10,000+ ticks/second calculation
- Replay playback: Smooth 60 FPS
- Part list render: < 100ms for 140+ parts
- Level editor: < 16ms per frame during manipulation
- IndexedDB operations: < 50ms for save/load
- Memory usage: < 200MB active session
- .MECHA file size: < 10KB average

---

## Testing Scenarios

### Builder Tests
- [ ] Equip valid part → shows in preview, updates stats
- [ ] Equip overweight build → shows warning, prevents save
- [ ] Remove brain chip → not allowed, chip is mandatory
- [ ] Change brain chip → updates AI behavior preview
- [ ] Export .MECHA → valid JSON file downloads
- [ ] Import .MECHA → loads correctly with new ID

### AI Configuration Tests
- [ ] Slider change → behavior description updates
- [ ] Brain chip swap → effect description changes
- [ ] Creative Mode → all chips unlocked and selectable
- [ ] Standard Mode → locked chips grayed out

### Battle Tests
- [ ] Same seed + mechs + arena → identical battle outcome
- [ ] Missile passes through building → hits target
- [ ] Bullet blocked by building → no damage
- [ ] Mech on water → visibly slower movement
- [ ] Mech on ice → momentum/sliding behavior
- [ ] Phoenix chip triggers → mech revives once
- [ ] Timeline scrub → state matches tick exactly

### Level Editor Tests
- [ ] Place spawn points → both required for valid arena
- [ ] Place building → collision bounds respected
- [ ] Paint water tile → movement penalty applied in test
- [ ] Export arena JSON → valid format
- [ ] Import arena JSON → loads correctly

### Tournament Tests
- [ ] Fetch opponents (online) → receives .MECHA data
- [ ] Fallback mode (offline) → uses bundled mechs
- [ ] Win tournament → new roster available
- [ ] Battle result → appears in mail with replay

### Replay Tests
- [ ] Open from mail → loads battle log correctly
- [ ] Speed controls → smooth playback at all speeds
- [ ] Timeline seek → accurate state reconstruction
- [ ] Brain chip activation → visual indicator shows

---

## Extended Features (Future Phases)

### Phase 2: Team Battles
- 2v2 and 3v3 formats
- Team AI coordination via shared behavior settings
- Combined weight limits across team

### Phase 3: Community Integration
- In-game .MECHA browser (curated from GitHub)
- Rating and favorites system
- "Clone and modify" workflow for community mechs

### Phase 4: Advanced Arenas
- Dynamic hazards (moving obstacles, timed events)
- Destructible environment elements
- Weather effects (rain reduces visibility, wind affects missiles)

### Phase 5: Spectator Features
- Shareable replay links
- Embedded replay viewer for external sites
- Commentary track support

---

## Offline-First Architecture

**Bundled Fallback Data:**
- 50+ pre-built .MECHA files for offline tournaments
- 10 default arenas covering all terrain types
- Complete part and brain chip definitions
- All SVG assets embedded

**Online Enhancement (When Available):**
- Fetch fresh opponent .MECHA data from server
- Expanded opponent variety based on global player pool
- Rank-based matchmaking from live database

**Graceful Degradation:**
- No features require online connectivity
- Online-only features show clear messaging when unavailable
- All progression works offline (syncs when online in future versions)

---

## Stigmergic Multiplayer Philosophy

Players never interact directly. Instead:

1. **Signal Emission:** Player completes a mech build (.MECHA file)
2. **Signal Storage:** .MECHA data archived on shared database
3. **Signal Reading:** Other players retrieve .MECHA data as opponents
4. **Emergent Evolution:** Successful designs propagate through the community
5. **Collective Intelligence:** The meta-game evolves through player experimentation

The database is a shared memory—a trace of every architect's work. The most successful mechs rise in rank, becoming more likely to be encountered. Failed designs fade into the archive. The game world learns and adapts without any direct player-to-player communication.

**This is multiplayer without multiplayer.**
