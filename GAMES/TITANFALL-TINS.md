# TITANCORE: Mecha Tactical Command

## Description

TITANCORE is a single-player tactical mecha combat game built as a React component-based application with SVG graphics. Inspired by Titanfall's pilot-titan dynamics, the game focuses on pre-mission preparation, loadout customization, and turn-based isometric combat where players command both infantry pilots and towering mecha units.

The game emphasizes strategic depth through equipment management, squad composition, and tactical positioning rather than real-time action. Players manage a mercenary company, equipping soldiers and mecha before deploying them on missions across procedurally-arranged battlefields.

The architecture prioritizes separation of concerns with small, focused component files to enable maintainable development and future multiplayer expansion. All graphics are static SVG with no CSS animations, relying on state transitions for visual feedback.

## Core Game Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                         GAME FLOW                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐ │
│   │  HANGAR  │───▶│  ARMORY  │───▶│ BRIEFING │───▶│  DEPLOY  │ │
│   │  (Base)  │    │ (Loadout)│    │ (Mission)│    │ (Combat) │ │
│   └──────────┘    └──────────┘    └──────────┘    └──────────┘ │
│        │                                               │        │
│        │              ┌──────────┐                     │        │
│        └──────────────│  DEBRIEF │◀────────────────────┘        │
│                       │ (Results)│                              │
│                       └──────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

### Game Screens

1. **Hangar Screen**: Company overview, pilot roster, mecha bay, mission selection
2. **Armory Screen**: Equipment inventory, loadout management for pilots and mecha
3. **Briefing Screen**: Mission objectives, terrain preview, squad deployment slots
4. **Deploy Screen**: Isometric tactical combat with turn-based movement and actions
5. **Debrief Screen**: Mission results, rewards, experience gains, salvage

## Functionality

### 1. Hangar System

The Hangar serves as the main hub displaying:

**Pilot Roster Panel**
- Grid of pilot cards showing: portrait (SVG avatar), name, rank, class icon, status (ready/injured/KIA)
- Click pilot to select for detail view
- Maximum 12 pilots in roster
- Injured pilots show recovery timer (missions remaining)

**Mecha Bay Panel**
- Grid of mecha cards showing: silhouette, designation, chassis class, status (operational/damaged/destroyed)
- Click mecha to select for detail view
- Maximum 8 mecha in bay
- Damaged mecha show repair cost

**Mission Board Panel**
- List of 3 available missions with: name, difficulty (1-5 stars), reward range, terrain type icon
- Missions refresh after completing any mission
- Click mission to proceed to Briefing

**Company Stats Bar**
- Credits balance
- Company reputation (affects mission availability)
- Current date/time (missions advance time)

### 2. Armory System

The Armory manages all equipment with two modes:

**Pilot Loadout Mode**

Each pilot has equipment slots:
- Primary Weapon (rifles, SMGs, shotguns, snipers)
- Secondary Weapon (pistols OR anti-mecha weapons - mutually exclusive choice)
- Tactical Ability (active cooldown ability)
- Ordnance (grenades, mines, deployables)
- Kit Slot 1 (passive bonus)
- Kit Slot 2 (passive bonus)

**Mecha Loadout Mode**

Each mecha has equipment slots:
- Primary Weapon System (locked to chassis in base game)
- Defensive System (shields, smoke, armor abilities)
- Utility System (sensors, boosters, support gear)
- Core Module (ultimate ability, locked to chassis)
- Kit Slot 1 (passive bonus)
- Kit Slot 2 (passive bonus)
- Kit Slot 3 (passive bonus)

**Inventory Panel**
- Tabbed view: All | Weapons | Abilities | Ordnance | Kits | Mecha Parts
- Each item shows: icon, name, stats summary, quantity owned
- Drag-and-drop to equipment slots
- Right-click for detailed stat comparison tooltip

**Equipment Comparison**
- When hovering item over slot, show stat delta (green +/red -)
- Stats include: damage, range, accuracy, weight, cooldown, special effects

### 3. Briefing System

Pre-mission planning interface:

**Mission Intel Panel**
- Mission name and narrative description
- Primary objective (required for success)
- Secondary objectives (bonus rewards)
- Enemy composition estimate (unit types, approximate count)
- Terrain hazards and features

**Deployment Planner**
- Shows deployment zone on miniature map
- Deployment slots: 4 pilot slots, 2 mecha slots
- Drag pilots/mecha from roster to slots
- Linked pilot-mecha pairs highlighted (pilot assigned to specific mecha)
- Unlinked pilots deploy on foot only

**Loadout Quick-Edit**
- Selected unit shows current loadout
- Quick-swap buttons for each slot
- "Edit in Armory" button for full customization

**Launch Button**
- Disabled until minimum deployment met (at least 2 pilots)
- Shows estimated mission duration
- Confirms and transitions to Deploy screen

### 4. Deploy System (Tactical Combat)

Turn-based isometric combat on hex-grid maps.

**Map Display**
- Isometric view with hex tiles
- Tile types: ground, elevated, water, hazard, cover, building interior
- Fog of war: unexplored tiles darkened
- Visible units rendered as SVG sprites

**Unit States**
- Pilots can be: On Foot, Embarked (in mecha), Ejecting, Downed
- Mecha can be: Operational, Damaged, Doomed, Destroyed
- State transitions have visual indicators (icon overlays)

**Turn Structure**
```
Player Phase:
  1. Select unit
  2. Move (within movement range)
  3. Action (attack, ability, interact, wait)
  4. Repeat for each unit
  5. End Turn

Enemy Phase:
  1. AI processes each enemy unit
  2. Animations/state updates shown sequentially
  3. Return to Player Phase

Round ends when all units on both sides have acted.
```

**Movement System**
- Pilots: 4 hex base movement, can enter buildings, climb elevations
- Mecha: 3 hex base movement, cannot enter buildings, destroys light cover
- Movement preview shows path and cost
- Tiles in range highlighted, out of range grayed

**Action System**
- Attack: Select weapon, select target in range, confirm
- Ability: Select tactical/defensive ability, select target if needed
- Interact: Objectives, terminals, embark/disembark mecha
- Wait: End unit turn, gain overwatch if weapon supports it

**Combat Resolution**
- Hit chance calculated: base accuracy ± range modifier ± cover modifier ± elevation modifier
- Damage rolls within weapon damage range
- Critical hits on natural high rolls (bonus damage)
- Displayed as floating damage numbers

**Pilot-Mecha Interactions**
- Embark: Pilot adjacent to friendly mecha can enter (costs full action)
- Disembark: Pilot exits mecha to adjacent hex (costs movement)
- Eject: Emergency exit when mecha doomed (launches pilot, mecha may explode)
- Titanfall: Call reinforcement mecha to valid drop zone (once per mission, if available)

**Victory/Defeat Conditions**
- Victory: Primary objective completed
- Defeat: All player units destroyed OR objective failed
- Partial: Some pilots KIA but mission complete (reduced rewards)

### 5. Debrief System

Post-mission results screen:

**Mission Summary**
- Outcome banner (Victory/Defeat/Pyrrhic Victory)
- Objectives checklist with completion status
- Time taken (turns elapsed)

**Casualty Report**
- Pilots: status changes (healthy→injured, injured→KIA)
- Mecha: damage sustained, repair costs
- Losses are permanent (KIA pilots removed from roster)

**Rewards Panel**
- Credits earned (base + objective bonuses)
- Salvage items (random equipment drops)
- Experience points per surviving unit
- Reputation change

**Continue Button**
- Returns to Hangar with updated state

### 6. Level Editor

Accessible from Hangar menu, creates custom maps for missions.

**Canvas Area**
- Isometric hex grid, resizable (min 10x10, max 30x30)
- Click to select tile, click again to cycle tile types
- Drag to paint multiple tiles

**Tile Palette**
- Ground (passable, no cover)
- Elevated (height +1, provides elevation bonus)
- Water (impassable to mecha, slows pilots)
- Hazard (damages units ending turn here)
- Light Cover (+20% defense, destructible)
- Heavy Cover (+40% defense, blocks movement)
- Building Wall (blocks all, pilots can breach)
- Building Interior (pilots only, full cover from outside)
- Objective Marker (mission target location)
- Spawn Zone Player (valid deployment tiles)
- Spawn Zone Enemy (AI unit placement)

**Tools Panel**
- Select Tool (click to select single tile)
- Paint Tool (drag to paint multiple)
- Fill Tool (flood fill connected same-type)
- Eraser Tool (reset to ground)
- Rotate View (45° isometric rotation)

**Map Properties**
- Map name (text input)
- Suggested difficulty (1-5 selector)
- Environment theme (affects color palette)
- Save / Load / Export buttons

**Validation**
- Must have at least 4 player spawn tiles
- Must have at least 1 objective marker
- Must have at least 6 enemy spawn tiles
- Shows error messages if invalid

## Technical Implementation

### Architecture Overview

```
src/
├── components/
│   ├── common/           # Shared UI components
│   │   ├── Button.jsx
│   │   ├── Panel.jsx
│   │   ├── Tooltip.jsx
│   │   ├── Modal.jsx
│   │   ├── ProgressBar.jsx
│   │   ├── StatBlock.jsx
│   │   └── IconBadge.jsx
│   │
│   ├── screens/          # Top-level screen containers
│   │   ├── HangarScreen.jsx
│   │   ├── ArmoryScreen.jsx
│   │   ├── BriefingScreen.jsx
│   │   ├── DeployScreen.jsx
│   │   ├── DebriefScreen.jsx
│   │   └── EditorScreen.jsx
│   │
│   ├── hangar/           # Hangar-specific components
│   │   ├── PilotRoster.jsx
│   │   ├── PilotCard.jsx
│   │   ├── MechaBay.jsx
│   │   ├── MechaCard.jsx
│   │   ├── MissionBoard.jsx
│   │   ├── MissionCard.jsx
│   │   └── CompanyStats.jsx
│   │
│   ├── armory/           # Armory-specific components
│   │   ├── LoadoutPanel.jsx
│   │   ├── EquipmentSlot.jsx
│   │   ├── InventoryGrid.jsx
│   │   ├── ItemCard.jsx
│   │   ├── StatComparison.jsx
│   │   └── LoadoutTabs.jsx
│   │
│   ├── briefing/         # Briefing-specific components
│   │   ├── MissionIntel.jsx
│   │   ├── DeploymentPlanner.jsx
│   │   ├── DeploymentSlot.jsx
│   │   ├── QuickLoadout.jsx
│   │   └── TerrainPreview.jsx
│   │
│   ├── deploy/           # Deploy/combat-specific components
│   │   ├── BattleMap.jsx
│   │   ├── HexTile.jsx
│   │   ├── UnitSprite.jsx
│   │   ├── ActionMenu.jsx
│   │   ├── TargetingOverlay.jsx
│   │   ├── TurnTracker.jsx
│   │   ├── CombatLog.jsx
│   │   ├── UnitInfoPanel.jsx
│   │   └── ObjectiveTracker.jsx
│   │
│   ├── debrief/          # Debrief-specific components
│   │   ├── MissionSummary.jsx
│   │   ├── CasualtyReport.jsx
│   │   ├── RewardsPanel.jsx
│   │   └── UnitExperience.jsx
│   │
│   ├── editor/           # Level editor components
│   │   ├── EditorCanvas.jsx
│   │   ├── TilePalette.jsx
│   │   ├── EditorTools.jsx
│   │   ├── MapProperties.jsx
│   │   └── ValidationPanel.jsx
│   │
│   └── svg/              # SVG asset components
│       ├── pilots/
│       │   ├── PilotAssault.jsx
│       │   ├── PilotSniper.jsx
│       │   ├── PilotEngineer.jsx
│       │   └── PilotSupport.jsx
│       ├── mecha/
│       │   ├── MechaStrider.jsx
│       │   ├── MechaAtlas.jsx
│       │   └── MechaOgre.jsx
│       ├── tiles/
│       │   ├── TileGround.jsx
│       │   ├── TileElevated.jsx
│       │   ├── TileWater.jsx
│       │   └── ... (other tile types)
│       ├── icons/
│       │   ├── WeaponIcons.jsx
│       │   ├── AbilityIcons.jsx
│       │   └── StatusIcons.jsx
│       └── ui/
│           ├── FramePanel.jsx
│           ├── FrameButton.jsx
│           └── FrameSlot.jsx
│
├── hooks/                # Custom React hooks
│   ├── useGameState.js
│   ├── useCombat.js
│   ├── usePathfinding.js
│   ├── useInventory.js
│   ├── useDragDrop.js
│   └── useLocalStorage.js
│
├── context/              # React context providers
│   ├── GameContext.jsx
│   ├── CombatContext.jsx
│   └── EditorContext.jsx
│
├── systems/              # Game logic (pure functions)
│   ├── combat/
│   │   ├── damageCalculation.js
│   │   ├── hitChanceCalculation.js
│   │   ├── abilityEffects.js
│   │   └── statusEffects.js
│   ├── movement/
│   │   ├── pathfinding.js
│   │   ├── movementCosts.js
│   │   └── lineOfSight.js
│   ├── ai/
│   │   ├── enemyBehavior.js
│   │   ├── targetPriority.js
│   │   └── tacticalPositioning.js
│   ├── economy/
│   │   ├── missionRewards.js
│   │   ├── repairCosts.js
│   │   └── salvageGeneration.js
│   └── progression/
│       ├── experienceSystem.js
│       ├── rankProgression.js
│       └── unlockSystem.js
│
├── data/                 # Static game data
│   ├── weapons.js
│   ├── abilities.js
│   ├── kits.js
│   ├── mechaChassisTypes.js
│   ├── pilotClasses.js
│   ├── enemyTypes.js
│   ├── missionTemplates.js
│   └── tileDefinitions.js
│
├── utils/                # Utility functions
│   ├── hexMath.js
│   ├── isometricTransform.js
│   ├── randomGeneration.js
│   └── saveLoad.js
│
└── App.jsx               # Root application component
```

### Data Models

#### Pilot

```javascript
{
  id: "pilot_001",                    // Unique identifier
  name: "Sarah Chen",                 // Display name
  callsign: "Viper",                  // Optional callsign
  class: "assault",                   // "assault" | "sniper" | "engineer" | "support"
  rank: 3,                            // 1-10, affects stat bonuses
  experience: 1250,                   // XP toward next rank
  status: "ready",                    // "ready" | "injured" | "kia"
  recoveryMissions: 0,                // Missions until recovered (if injured)
  stats: {
    health: 100,                      // Base HP
    movement: 4,                      // Hex movement range
    accuracy: 75,                     // Base hit chance percentage
    evasion: 15,                      // Dodge chance percentage
    armor: 0                          // Damage reduction
  },
  loadout: {
    primaryWeapon: "weapon_rifle_01",
    secondaryWeapon: "weapon_pistol_01",
    tactical: "ability_stim",
    ordnance: "ordnance_frag",
    kit1: "kit_fast_reload",
    kit2: null
  },
  linkedMechaId: "mecha_001",         // Assigned mecha or null
  portrait: "portrait_female_01"      // SVG portrait reference
}
```

#### Mecha

```javascript
{
  id: "mecha_001",
  designation: "TN-7734",             // Unit designation
  nickname: "Ironclad",               // Player-assigned name
  chassis: "atlas",                   // "strider" | "atlas" | "ogre"
  status: "operational",              // "operational" | "damaged" | "doomed" | "destroyed"
  currentHealth: 10000,
  maxHealth: 10000,
  repairCost: 0,                      // Credits to fully repair
  stats: {
    health: 10000,                    // Base HP (chassis-dependent)
    movement: 3,                      // Hex movement range
    armor: 25,                        // Damage reduction percentage
    coreCharge: 0,                    // 0-100, ultimate ability meter
    dashes: 1                         // Dash charges (chassis-dependent)
  },
  loadout: {
    primaryWeapon: "mecha_weapon_chaingun",  // Locked to chassis
    defensive: "mecha_def_vortex",
    utility: "mecha_util_sensor",
    coreModule: "mecha_core_salvo",          // Locked to chassis
    kit1: "mecha_kit_overcore",
    kit2: "mecha_kit_turbo",
    kit3: null
  },
  linkedPilotId: "pilot_001"
}
```

#### Equipment Item

```javascript
{
  id: "weapon_rifle_01",
  name: "R-201 Carbine",
  type: "weapon",                     // "weapon" | "ability" | "ordnance" | "kit" | "mecha_weapon" | etc.
  slot: "primaryWeapon",              // Which slot it fits
  category: "rifle",                  // Subcategory for filtering
  stats: {
    damage: [25, 35],                 // Min-max damage range
    range: 8,                         // Hex range
    accuracy: 85,                     // Base accuracy modifier
    rateOfFire: 3,                    // Attacks per action
    ammo: 24,                         // Shots before reload
    reloadTime: 1,                    // Actions to reload
    weight: 2                         // Affects movement
  },
  specialEffects: [],                 // Array of effect IDs
  description: "Reliable automatic rifle with consistent performance.",
  icon: "icon_rifle_01"
}
```

#### Mission

```javascript
{
  id: "mission_001",
  name: "Operation Iron Harvest",
  description: "Secure the abandoned factory complex before enemy reinforcements arrive.",
  difficulty: 3,                      // 1-5 stars
  terrain: "urban",                   // "urban" | "forest" | "industrial" | "arctic" | "desert"
  objectives: {
    primary: {
      type: "capture",                // "capture" | "eliminate" | "escort" | "survive" | "extract"
      target: "objective_factory",
      description: "Capture the central control room"
    },
    secondary: [
      {
        type: "eliminate",
        target: "enemy_commander",
        description: "Eliminate the enemy commander",
        reward: { credits: 500 }
      }
    ]
  },
  enemies: {
    grunts: 8,
    spectres: 4,
    pilots: 2,
    mecha: 1
  },
  rewards: {
    credits: [1500, 2500],            // Min-max range
    reputation: 10,
    salvageChance: 0.6
  },
  mapId: "map_urban_01",
  turnLimit: 20                       // Optional turn limit
}
```

#### Hex Tile

```javascript
{
  q: 5,                               // Axial coordinate q
  r: 3,                               // Axial coordinate r
  type: "ground",                     // Tile type string
  elevation: 0,                       // Height level (0-2)
  cover: "none",                      // "none" | "light" | "heavy" | "full"
  passablePilot: true,
  passableMecha: true,
  movementCost: 1,                    // Movement points to enter
  contents: {                         // What's on the tile
    unit: null,                       // Unit ID or null
    objective: null,                  // Objective ID or null
    hazard: null                      // Hazard type or null
  },
  visibility: "hidden"                // "hidden" | "revealed" | "visible"
}
```

#### Game State

```javascript
{
  screen: "hangar",                   // Current screen
  company: {
    name: "Iron Wolves",
    credits: 15000,
    reputation: 45,
    currentDate: "2185-03-15"
  },
  pilots: [ /* array of Pilot objects */ ],
  mecha: [ /* array of Mecha objects */ ],
  inventory: [ /* array of { itemId, quantity } */ ],
  unlockedItems: [ /* array of item IDs */ ],
  availableMissions: [ /* array of Mission objects */ ],
  currentMission: null,               // Active mission or null
  combatState: null,                  // Combat state when in Deploy screen
  customMaps: [ /* array of custom map data */ ],
  settings: {
    musicVolume: 0.7,
    sfxVolume: 0.8,
    autoEndTurn: false,
    confirmActions: true
  }
}
```

#### Combat State

```javascript
{
  missionId: "mission_001",
  map: [ /* 2D array of HexTile objects */ ],
  units: {
    player: [ /* array of deployed unit states */ ],
    enemy: [ /* array of enemy unit states */ ]
  },
  turnNumber: 1,
  phase: "player",                    // "player" | "enemy"
  activeUnitId: null,
  selectedAction: null,
  actionHistory: [ /* array of action records */ ],
  objectiveStatus: {
    primary: "incomplete",            // "incomplete" | "complete" | "failed"
    secondary: [ /* array of completion booleans */ ]
  }
}
```

### Isometric Hex Rendering

The game uses axial coordinates (q, r) for hex math and converts to screen coordinates for rendering.

**Coordinate Conversion:**

```javascript
// Constants for hex geometry
const HEX_SIZE = 32;  // Pixels from center to corner
const HEX_WIDTH = HEX_SIZE * 2;
const HEX_HEIGHT = Math.sqrt(3) * HEX_SIZE;

// Axial to screen (flat-top isometric)
function hexToScreen(q, r) {
  const x = HEX_SIZE * (3/2 * q);
  const y = HEX_SIZE * (Math.sqrt(3)/2 * q + Math.sqrt(3) * r);
  
  // Apply isometric transformation
  const isoX = (x - y) * Math.cos(Math.PI / 6);
  const isoY = (x + y) * Math.sin(Math.PI / 6) * 0.5;
  
  return { x: isoX, y: isoY };
}

// Screen to axial (for click detection)
function screenToHex(screenX, screenY) {
  // Reverse isometric transformation first
  // Then convert to axial coordinates
  // Round to nearest hex
}
```

**Rendering Order:**
1. Render tiles back-to-front (by r, then q descending)
2. Render tile decorations (cover, objectives)
3. Render units (sorted by position for proper overlap)
4. Render overlays (movement range, targeting)
5. Render UI elements (floating damage, status icons)

### Pathfinding System

A* algorithm adapted for hex grids:

```javascript
function findPath(startHex, endHex, unitType, map) {
  // Returns array of hex coordinates from start to end
  // Considers movement costs, passability based on unitType
  // Returns null if no valid path exists
}

function getMovementRange(unitHex, movementPoints, unitType, map) {
  // Returns Set of reachable hex coordinates
  // Uses Dijkstra's algorithm with movement costs
  // Respects unit type passability rules
}

function getLineOfSight(fromHex, toHex, map) {
  // Returns true if clear line of sight exists
  // Uses hex line drawing algorithm
  // Blocked by full cover tiles and elevation differences
}
```

### Combat Calculations

**Hit Chance:**
```javascript
function calculateHitChance(attacker, target, weapon, map) {
  let chance = weapon.stats.accuracy;
  
  // Range modifier: -5% per hex beyond optimal
  const distance = hexDistance(attacker.position, target.position);
  const optimalRange = Math.floor(weapon.stats.range * 0.6);
  if (distance > optimalRange) {
    chance -= (distance - optimalRange) * 5;
  }
  
  // Cover modifier
  const cover = getCoverBetween(attacker.position, target.position, map);
  if (cover === "light") chance -= 20;
  if (cover === "heavy") chance -= 40;
  
  // Elevation modifier
  const elevationDiff = getElevation(attacker.position) - getElevation(target.position);
  chance += elevationDiff * 10;  // Bonus for high ground
  
  // Target evasion
  chance -= target.stats.evasion;
  
  // Attacker accuracy stat
  chance += (attacker.stats.accuracy - 75);  // Relative to base 75
  
  return Math.max(5, Math.min(95, chance));  // Clamp 5-95%
}
```

**Damage Calculation:**
```javascript
function calculateDamage(weapon, target, isCritical) {
  const [minDamage, maxDamage] = weapon.stats.damage;
  let damage = randomInRange(minDamage, maxDamage);
  
  if (isCritical) {
    damage *= 1.5;
  }
  
  // Apply armor reduction
  const armorReduction = target.stats.armor / 100;
  damage *= (1 - armorReduction);
  
  return Math.round(damage);
}
```

### AI System

Enemy AI uses behavior trees with tactical considerations:

**Priority Order:**
1. Self-preservation (retreat if low health and able)
2. Objective defense (if guarding objective)
3. Target priority (damaged units > isolated units > high-threat units)
4. Positioning (seek cover, avoid clustering)

**Decision Making:**
```javascript
function determineEnemyAction(unit, combatState) {
  const possibleActions = [
    evaluateAttackOptions(unit, combatState),
    evaluateMovementOptions(unit, combatState),
    evaluateAbilityOptions(unit, combatState)
  ];
  
  // Score each action based on tactical value
  // Return highest-scored valid action
}
```

### State Management

Uses React Context with useReducer for predictable state updates:

```javascript
// GameContext.jsx
const GameContext = createContext();

const gameReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_PILOT':
      return { ...state, pilots: updatePilot(state.pilots, action.payload) };
    case 'UPDATE_MECHA':
      return { ...state, mecha: updateMecha(state.mecha, action.payload) };
    case 'ADD_ITEM':
      return { ...state, inventory: addItem(state.inventory, action.payload) };
    case 'SET_SCREEN':
      return { ...state, screen: action.payload };
    case 'START_MISSION':
      return { ...state, currentMission: action.payload, screen: 'deploy' };
    case 'END_MISSION':
      return processMissionEnd(state, action.payload);
    // ... other actions
  }
};
```

**Persistence:**
- Auto-save to localStorage after each significant action
- Manual save/load slots (3 slots)
- Export/import as JSON for backup

### Multiplayer Considerations

While single-player only for initial release, architecture supports future multiplayer:

- All game state is serializable JSON
- Actions are discrete and can be transmitted as messages
- Combat state can be validated server-side
- Turn-based structure allows async play
- No client-side randomness (seeds transmitted with actions)

## Style Guide

### Color Palette

```
Primary UI:
  --color-bg-dark: #0a0e17
  --color-bg-panel: #141b29
  --color-bg-hover: #1e2a3d
  --color-border: #2a3a52
  --color-border-active: #4a7c9b

Text:
  --color-text-primary: #e8ecf1
  --color-text-secondary: #8899a8
  --color-text-muted: #506070

Faction Colors:
  --color-player: #4a9eff
  --color-enemy: #ff4a4a
  --color-neutral: #88888

Status Colors:
  --color-health: #4aff6a
  --color-damage: #ff4a4a
  --color-shield: #4adfff
  --color-warning: #ffaa4a
  --color-xp: #bb88ff

Terrain Colors:
  --color-ground: #3a4a3a
  --color-water: #2a4a6a
  --color-hazard: #6a3a2a
  --color-cover: #4a4a3a
```

### Typography

- Headings: System UI, bold, sizes 24/20/16px
- Body: System UI, regular, 14px
- Data: Monospace, 12px (for stats, coordinates)
- All text should have subtle text-shadow for readability

### SVG Style

- All graphics use flat shading with 2-3 value ranges
- Black outlines (1-2px stroke) for definition
- Consistent lighting direction (top-left)
- Limited palette per sprite (5-8 colors max)
- Mecha use angular, mechanical shapes
- Pilots use rounded, organic shapes
- Icons are 24x24 or 32x32 viewBox

### Panel Design

- Panels have 2px border with corner accents
- Slight inner glow on active/selected panels
- Headers are slightly lighter background
- Consistent 8px internal padding
- 4px gaps between elements

## Testing Scenarios

### Scenario 1: Complete Game Loop
1. Start new game with default company
2. Select a difficulty 1 mission
3. Deploy minimum 2 pilots (no mecha)
4. Complete primary objective
5. Return to hangar with rewards applied
6. Verify credits increased, XP gained

### Scenario 2: Loadout Management
1. Open Armory from Hangar
2. Select a pilot
3. Drag weapon from inventory to primary slot
4. Verify stat comparison appears
5. Confirm equip, verify inventory quantity decreased
6. Unequip item, verify returns to inventory

### Scenario 3: Pilot-Mecha Link
1. Have pilot with linked mecha
2. Deploy both to mission
3. Start pilot near mecha
4. Execute Embark action
5. Verify pilot removed from map, mecha gains pilot bonuses
6. Execute Disembark action
7. Verify pilot placed adjacent to mecha

### Scenario 4: Combat Resolution
1. Position pilot with clear shot at enemy
2. Select Attack action
3. Verify hit chance displayed correctly
4. Execute attack
5. Verify damage applied (or miss registered)
6. Verify enemy health bar updated
7. If killed, verify unit removed and XP/core charge gained

### Scenario 5: Level Editor Workflow
1. Open Level Editor
2. Create 15x15 map
3. Paint various terrain types
4. Place spawn zones and objectives
5. Run validation
6. Save map
7. Verify map appears in custom mission list
8. Load map in new session

### Scenario 6: Permadeath and Injury
1. Deploy pilot to dangerous mission
2. Allow pilot to take lethal damage
3. Verify pilot status changes to KIA
4. Complete mission
5. Verify pilot removed from roster in debrief
6. Test injury (non-lethal incapacitation)
7. Verify injured pilot unavailable for missions until recovered

## Accessibility Requirements

- All interactive elements keyboard accessible
- Tab order follows logical flow
- Focus indicators visible on all focusable elements
- Color is not sole indicator of state (icons/patterns supplement)
- Minimum contrast ratio 4.5:1 for text
- Screen reader labels on all buttons and interactive elements
- Pause/resume capability (turn-based nature helps)
- Tooltips accessible via keyboard focus
- Font sizes adjustable via browser zoom (relative units)

## Performance Goals

- Initial load under 3 seconds
- Screen transitions under 200ms
- Combat action resolution under 100ms
- No jank during hex grid rendering (up to 30x30 = 900 hexes)
- Smooth drag-and-drop at 60fps
- Save/load operations under 500ms
- Memory usage under 100MB

### Optimization Strategies

- SVG sprites loaded once and reused via `<use>` references
- Hex grid uses virtualization for large maps (only render visible)
- React.memo on expensive components (HexTile, UnitSprite)
- useMemo for pathfinding results
- Debounce frequent state updates (drag position)
- Web Workers for AI calculations on enemy turn

## Extended Features (Future Versions)

### Version 1.1 - Campaign Mode
- Linked mission sequences with narrative
- Persistent pilot relationships
- Branching story paths
- Boss encounters

### Version 1.2 - Advanced Customization
- Mecha paint schemes
- Pilot skill trees
- Weapon modifications
- Company upgrades (facilities)

### Version 1.3 - Multiplayer Foundation
- Hot-seat local multiplayer
- Custom skirmish mode
- Replay system
- Leaderboards for custom maps

### Version 1.4 - Online Multiplayer
- Peer-to-peer matches
- Ranked ladder
- Async play-by-mail mode
- Spectator mode

## Implementation Phases

### Phase 1: Core Framework
1. Project setup with Vite + React
2. Implement common UI components
3. Create SVG asset system
4. Implement GameContext and state management
5. Build screen navigation system

### Phase 2: Hangar & Armory
1. Hangar screen layout
2. Pilot roster with mock data
3. Mecha bay with mock data
4. Mission board with mock data
5. Armory screen with drag-drop loadout
6. Inventory system

### Phase 3: Combat Foundation
1. Hex grid rendering (isometric)
2. Hex math utilities
3. Unit placement and display
4. Click-to-select mechanics
5. Movement range calculation
6. Pathfinding implementation

### Phase 4: Combat Actions
1. Action menu system
2. Attack action with targeting
3. Hit/damage calculations
4. Unit death handling
5. Turn structure (player/enemy phases)
6. Objective tracking

### Phase 5: AI & Polish
1. Enemy AI decision making
2. Ability system
3. Pilot-mecha interactions
4. Combat log
5. Victory/defeat conditions
6. Debrief screen

### Phase 6: Level Editor
1. Editor canvas and controls
2. Tile palette
3. Paint/fill tools
4. Map validation
5. Save/load custom maps
6. Integration with mission system

### Phase 7: Game Balance & Content
1. Balance pass on all stats
2. Add full weapon variety
3. Add full ability variety
4. Create 10+ mission templates
5. Tutorial mission
6. New game setup flow

### Phase 8: Polish & Launch
1. Sound effects (optional)
2. Accessibility audit
3. Performance optimization
4. Bug fixes
5. Save system hardening
6. Final testing

---

*TINS Specification v1.0 - TITANCORE: Mecha Tactical Command*
*This document contains all information needed to generate a complete implementation.*
