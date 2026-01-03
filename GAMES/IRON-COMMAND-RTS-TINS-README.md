# Iron Command: React RTS Game

A real-time strategy game inspired by Command & Conquer: Red Alert, built with React, Three.js, and @react-three/fiber. Features isometric 3D wireframe graphics with vertex shading, eliminating the need for texture assets. Includes a comprehensive level editor for community map creation with JSON export.

## Description

Iron Command delivers an authentic RTS experience through a modern web stack. The game implements cell-based terrain, sidebar construction UI, asymmetric faction balance, fog of war, and full economic/combat systems derived from the Red Alert design blueprint. The wireframe aesthetic with per-vertex coloring creates a distinctive visual style while remaining performant across devices.

The architecture prioritizes separation of concerns through a modular component system. Each game subsystem (terrain, units, buildings, UI, AI) exists as an independent module with clearly defined interfaces. This enables focused development on specific areas without cascading changes, and positions the codebase for future multiplayer implementation through deterministic state management.

## Technical Stack

```
Framework:      React 18+ with functional components and hooks
3D Rendering:   Three.js via @react-three/fiber and @react-three/drei
State:          Zustand for game state, React Context for UI state
Styling:        Tailwind CSS for UI components
Data Format:    JSON for maps, units, buildings, and game configuration
Build:          Vite with TypeScript support
```

## Project Structure

```
src/
├── components/
│   ├── game/
│   │   ├── GameCanvas.jsx          # Main Three.js canvas wrapper
│   │   ├── IsometricCamera.jsx     # Camera controls and positioning
│   │   ├── SelectionBox.jsx        # Marquee selection rendering
│   │   └── CursorManager.jsx       # Context-sensitive cursor states
│   │
│   ├── terrain/
│   │   ├── TerrainGrid.jsx         # Cell-based terrain mesh generation
│   │   ├── TerrainCell.jsx         # Individual cell with vertex colors
│   │   ├── ResourceField.jsx       # Ore/gem deposit visualization
│   │   ├── WaterPlane.jsx          # Animated water surface
│   │   └── FogOfWar.jsx            # Shroud and fog overlay system
│   │
│   ├── entities/
│   │   ├── UnitRenderer.jsx        # Unit mesh with faction colors
│   │   ├── BuildingRenderer.jsx    # Structure mesh with damage states
│   │   ├── ProjectileRenderer.jsx  # Weapon projectile visualization
│   │   ├── EffectRenderer.jsx      # Explosions, muzzle flash, etc.
│   │   └── SelectionRing.jsx       # Unit/building selection indicator
│   │
│   ├── ui/
│   │   ├── Sidebar.jsx             # Main sidebar container
│   │   ├── CreditsDisplay.jsx      # Real-time credits counter
│   │   ├── PowerBar.jsx            # Power production/consumption meter
│   │   ├── Minimap.jsx             # Radar minimap with click-to-move
│   │   ├── BuildTabs.jsx           # Category tab selector
│   │   ├── BuildGrid.jsx           # Production button grid
│   │   ├── BuildButton.jsx         # Individual build item with progress
│   │   ├── CommandPanel.jsx        # Selected unit commands
│   │   ├── UnitInfoPanel.jsx       # Selected unit stats display
│   │   └── GameMenu.jsx            # Pause menu and options
│   │
│   └── editor/
│       ├── LevelEditor.jsx         # Main editor container
│       ├── TerrainBrush.jsx        # Terrain painting tools
│       ├── EntityPlacer.jsx        # Unit/building placement
│       ├── TriggerEditor.jsx       # Mission trigger configuration
│       ├── PropertyPanel.jsx       # Selected object properties
│       └── EditorToolbar.jsx       # Tool selection and file ops
│
├── systems/
│   ├── GameLoop.js                 # Core game tick and update cycle
│   ├── PathfindingSystem.js        # A* with terrain cost modifiers
│   ├── CombatSystem.js             # Damage calculation and targeting
│   ├── ProductionSystem.js         # Build queue and prerequisites
│   ├── EconomySystem.js            # Credits, storage, harvesting
│   ├── PowerSystem.js              # Power grid management
│   ├── VisionSystem.js             # Fog of war and sight ranges
│   ├── SelectionSystem.js          # Unit selection and control groups
│   ├── CommandSystem.js            # Order processing and unit AI
│   └── AISystem.js                 # Computer opponent behavior
│
├── stores/
│   ├── gameStore.js                # Main game state (Zustand)
│   ├── uiStore.js                  # UI state and preferences
│   └── editorStore.js              # Level editor state
│
├── data/
│   ├── factions/
│   │   ├── allied.json             # Allied faction definition
│   │   └── soviet.json             # Soviet faction definition
│   ├── units/
│   │   ├── infantry.json           # Infantry unit definitions
│   │   ├── vehicles.json           # Vehicle unit definitions
│   │   ├── aircraft.json           # Aircraft unit definitions
│   │   └── naval.json              # Naval unit definitions
│   ├── buildings/
│   │   ├── production.json         # Production structure definitions
│   │   ├── defense.json            # Defensive structure definitions
│   │   └── support.json            # Support structure definitions
│   ├── weapons/
│   │   └── weapons.json            # Weapon and warhead definitions
│   ├── terrain/
│   │   └── terrainTypes.json       # Terrain type definitions
│   └── maps/
│       └── [mapName].json          # Individual map files
│
├── hooks/
│   ├── useGameLoop.js              # Game loop subscription
│   ├── useSelection.js             # Selection state and handlers
│   ├── useCamera.js                # Camera controls
│   ├── useKeyboard.js              # Keyboard shortcut handling
│   └── useBuildPlacement.js        # Building placement preview
│
├── utils/
│   ├── geometry.js                 # Isometric coordinate math
│   ├── pathfinding.js              # A* algorithm implementation
│   ├── random.js                   # Seeded random for determinism
│   └── wireframeMesh.js            # Vertex-colored mesh generation
│
└── App.jsx                         # Root application component
```

## Data Models

### Map Format (JSON)

```json
{
  "meta": {
    "name": "Battle for Control",
    "author": "MapMaker",
    "version": "1.0.0",
    "description": "A 2-player skirmish map with central ore fields",
    "players": 2,
    "size": { "width": 64, "height": 64 }
  },
  "terrain": {
    "cells": [
      {
        "x": 0, "y": 0,
        "type": "clear",
        "elevation": 0,
        "ore": 0,
        "passability": { "foot": 1.0, "track": 1.0, "wheel": 1.0, "float": 0, "air": 1.0 }
      }
    ],
    "overlays": [
      { "x": 10, "y": 10, "type": "road", "variant": "straight_ns" },
      { "x": 15, "y": 20, "type": "bridge", "variant": "wood_ew", "health": 400 }
    ]
  },
  "resources": [
    { "x": 12, "y": 8, "type": "ore", "density": 10, "regenerates": true },
    { "x": 50, "y": 45, "type": "gems", "density": 5, "regenerates": false }
  ],
  "startPositions": [
    { "player": 0, "x": 5, "y": 5, "facing": 135 },
    { "player": 1, "x": 58, "y": 58, "facing": 315 }
  ],
  "preplacedEntities": [
    { "type": "building", "id": "oilDerrick", "x": 32, "y": 32, "owner": "neutral", "capturable": true }
  ],
  "triggers": [
    {
      "id": "reinforcements",
      "condition": { "type": "timeElapsed", "seconds": 300 },
      "action": { "type": "spawnUnits", "units": ["mediumTank", "mediumTank"], "at": { "x": 0, "y": 32 }, "owner": 0 }
    }
  ],
  "options": {
    "startingCredits": 10000,
    "superweaponsEnabled": true,
    "cratesEnabled": true,
    "shroudEnabled": true
  }
}
```

### Unit Definition (JSON)

```json
{
  "id": "mediumTank",
  "name": "Medium Tank",
  "faction": "allied",
  "category": "vehicle",
  "cost": 800,
  "buildTime": 38.4,
  "prerequisites": ["warFactory"],
  "stats": {
    "health": 400,
    "armor": "heavy",
    "speed": 8,
    "turnRate": 5,
    "sightRange": 6,
    "crushable": false,
    "canCrush": true,
    "selfHealing": null
  },
  "movement": {
    "type": "track",
    "terrain": {
      "clear": 1.0,
      "road": 1.0,
      "rough": 0.6,
      "water": 0,
      "ore": 1.0
    }
  },
  "weapons": [
    {
      "id": "105mmCannon",
      "slot": "primary",
      "turret": true,
      "turretRotationSpeed": 4
    }
  ],
  "model": {
    "meshId": "tank_medium",
    "scale": 1.0,
    "turretOffset": { "x": 0, "y": 0.5, "z": 0 }
  },
  "sounds": {
    "select": "vehicle_select",
    "move": "vehicle_move",
    "attack": "cannon_fire"
  }
}
```

### Building Definition (JSON)

```json
{
  "id": "warFactory",
  "name": "War Factory",
  "faction": "both",
  "category": "production",
  "cost": 2000,
  "buildTime": 96,
  "prerequisites": ["oreRefinery"],
  "stats": {
    "health": 1000,
    "armor": "concrete",
    "sightRange": 5
  },
  "footprint": {
    "width": 3,
    "height": 3,
    "cells": [
      [1,1,1],
      [1,1,1],
      [1,1,1]
    ],
    "exitPoint": { "x": 1, "y": 3 }
  },
  "power": {
    "production": 0,
    "consumption": 30
  },
  "production": {
    "type": "vehicle",
    "rallyPoint": true,
    "queueSize": 5
  },
  "unlocks": [
    "harvester", "lightTank", "mediumTank", "apc", "artilleryTank", "mcv"
  ],
  "model": {
    "meshId": "building_warfactory",
    "scale": 1.0,
    "animatedParts": ["door"]
  }
}
```

### Weapon Definition (JSON)

```json
{
  "id": "105mmCannon",
  "name": "105mm Cannon",
  "damage": 40,
  "rateOfFire": 50,
  "range": 5,
  "projectile": {
    "type": "ballistic",
    "speed": 20,
    "meshId": "projectile_shell"
  },
  "warhead": {
    "id": "apShell",
    "type": "kinetic",
    "verses": {
      "none": 100,
      "flak": 75,
      "plate": 90,
      "light": 100,
      "medium": 90,
      "heavy": 75,
      "wood": 100,
      "steel": 60,
      "concrete": 50,
      "special_1": 100,
      "special_2": 100
    },
    "cellSpread": 0,
    "percentAtMax": 1.0
  },
  "targeting": {
    "canTarget": ["ground", "structure"],
    "cannotTarget": ["air", "underwater"],
    "autoAcquire": true,
    "leadTarget": true
  }
}
```

### Faction Definition (JSON)

```json
{
  "id": "allied",
  "name": "Allies",
  "color": { "primary": "#3B82F6", "secondary": "#1D4ED8" },
  "startingUnits": ["mcv"],
  "techTree": {
    "constructionYard": {
      "unlocks": ["powerPlant", "barracks", "oreRefinery"]
    },
    "powerPlant": {
      "prerequisites": ["constructionYard"],
      "unlocks": ["advancedPowerPlant"]
    },
    "barracks": {
      "prerequisites": ["constructionYard", "powerPlant"],
      "unlocks": ["rifleman", "rocketSoldier", "engineer", "spy", "thief"]
    },
    "oreRefinery": {
      "prerequisites": ["constructionYard", "powerPlant"],
      "unlocks": ["radarDome", "warFactory"]
    },
    "warFactory": {
      "prerequisites": ["oreRefinery"],
      "unlocks": ["harvester", "lightTank", "mediumTank", "apc", "artilleryTank", "mcv"]
    },
    "radarDome": {
      "prerequisites": ["oreRefinery"],
      "unlocks": ["techCenter"]
    },
    "techCenter": {
      "prerequisites": ["radarDome"],
      "unlocks": ["chronosphere", "gapGenerator", "gpsSatellite", "cruiser"]
    }
  },
  "superweapons": [
    {
      "id": "chronosphere",
      "building": "chronosphere",
      "chargeTime": 420,
      "effect": "teleport"
    },
    {
      "id": "gpsSatellite",
      "building": "gpsSatellite",
      "chargeTime": 300,
      "effect": "revealMap"
    }
  ]
}
```

## Game State Structure

```javascript
// Zustand store structure
const gameState = {
  // Meta
  tick: 0,
  gameSpeed: 1.0,
  isPaused: false,
  gameMode: 'skirmish', // 'campaign' | 'skirmish' | 'editor'
  
  // Map
  map: {
    width: 64,
    height: 64,
    cells: [], // 2D array of cell objects
    resources: [], // Resource field objects
  },
  
  // Players
  players: [
    {
      id: 0,
      faction: 'allied',
      color: '#3B82F6',
      credits: 10000,
      creditStorage: 2000,
      powerProduction: 0,
      powerConsumption: 0,
      isHuman: true,
      defeated: false,
    }
  ],
  
  // Entities
  entities: {
    units: new Map(), // id -> unit object
    buildings: new Map(), // id -> building object
    projectiles: new Map(), // id -> projectile object
  },
  
  // Production queues (per player, per production type)
  production: {
    0: { // player 0
      structure: { queue: [], progress: 0, paused: false },
      infantry: { queue: [], progress: 0, paused: false },
      vehicle: { queue: [], progress: 0, paused: false },
      aircraft: { queue: [], progress: 0, paused: false },
      naval: { queue: [], progress: 0, paused: false },
    }
  },
  
  // Vision
  vision: {
    0: { // player 0
      explored: new Set(), // Set of "x,y" strings
      visible: new Set(), // Currently visible cells
    }
  },
  
  // Selection
  selection: {
    units: [],
    buildings: [],
    controlGroups: { 0: [], 1: [], 2: [], 3: [], 4: [], 5: [], 6: [], 7: [], 8: [], 9: [] },
  },
  
  // Camera
  camera: {
    x: 0,
    y: 0,
    zoom: 1.0,
  },
  
  // Actions
  actions: {
    setGameSpeed: (speed) => {},
    togglePause: () => {},
    selectUnits: (unitIds, additive) => {},
    issueOrder: (unitIds, order) => {},
    startProduction: (playerId, itemId) => {},
    cancelProduction: (playerId, type, index) => {},
    placeBuilding: (playerId, buildingId, x, y) => {},
    sellBuilding: (buildingId) => {},
    repairBuilding: (buildingId) => {},
  },
};
```

## Core Systems Implementation

### Game Loop (GameLoop.js)

The game loop runs at a fixed tick rate (15 ticks per second) for deterministic simulation, decoupled from render frame rate.

```javascript
const TICK_RATE = 15; // Ticks per second
const TICK_DURATION = 1000 / TICK_RATE; // ~66.67ms

function createGameLoop(store) {
  let lastTick = performance.now();
  let accumulator = 0;
  let running = true;

  function loop(currentTime) {
    if (!running) return;
    
    const deltaTime = currentTime - lastTick;
    lastTick = currentTime;
    accumulator += deltaTime;
    
    while (accumulator >= TICK_DURATION) {
      if (!store.getState().isPaused) {
        processTick(store);
      }
      accumulator -= TICK_DURATION;
    }
    
    requestAnimationFrame(loop);
  }
  
  function processTick(store) {
    const state = store.getState();
    const tick = state.tick + 1;
    
    // Update systems in deterministic order
    updateProduction(state, tick);
    updateEconomy(state, tick);
    updatePower(state, tick);
    updateVision(state, tick);
    updateUnitAI(state, tick);
    updateMovement(state, tick);
    updateCombat(state, tick);
    updateProjectiles(state, tick);
    cleanupDeadEntities(state);
    
    store.setState({ tick });
  }
  
  return {
    start: () => { running = true; requestAnimationFrame(loop); },
    stop: () => { running = false; },
  };
}
```

### Pathfinding System (PathfindingSystem.js)

A* pathfinding with terrain cost modifiers and unit-type-specific passability.

```javascript
function findPath(startX, startY, endX, endY, unitType, map) {
  const openSet = new PriorityQueue();
  const closedSet = new Set();
  const cameFrom = new Map();
  const gScore = new Map();
  const fScore = new Map();
  
  const startKey = `${startX},${startY}`;
  const endKey = `${endX},${endY}`;
  
  gScore.set(startKey, 0);
  fScore.set(startKey, heuristic(startX, startY, endX, endY));
  openSet.enqueue(startKey, fScore.get(startKey));
  
  while (!openSet.isEmpty()) {
    const currentKey = openSet.dequeue();
    
    if (currentKey === endKey) {
      return reconstructPath(cameFrom, currentKey);
    }
    
    closedSet.add(currentKey);
    const [cx, cy] = currentKey.split(',').map(Number);
    
    for (const [nx, ny] of getNeighbors(cx, cy, map)) {
      const neighborKey = `${nx},${ny}`;
      if (closedSet.has(neighborKey)) continue;
      
      const cell = map.cells[ny][nx];
      const moveCost = getMovementCost(cell, unitType);
      if (moveCost === Infinity) continue; // Impassable
      
      const tentativeG = gScore.get(currentKey) + moveCost;
      
      if (!gScore.has(neighborKey) || tentativeG < gScore.get(neighborKey)) {
        cameFrom.set(neighborKey, currentKey);
        gScore.set(neighborKey, tentativeG);
        fScore.set(neighborKey, tentativeG + heuristic(nx, ny, endX, endY));
        
        if (!openSet.contains(neighborKey)) {
          openSet.enqueue(neighborKey, fScore.get(neighborKey));
        }
      }
    }
  }
  
  return null; // No path found
}

function getMovementCost(cell, unitType) {
  const passability = cell.passability[unitType.movement.type];
  if (passability === 0) return Infinity;
  
  const terrainModifier = unitType.movement.terrain[cell.type] || 1.0;
  if (terrainModifier === 0) return Infinity;
  
  return 1.0 / (passability * terrainModifier);
}

function heuristic(x1, y1, x2, y2) {
  // Diagonal distance heuristic
  const dx = Math.abs(x2 - x1);
  const dy = Math.abs(y2 - y1);
  return Math.max(dx, dy) + (Math.SQRT2 - 1) * Math.min(dx, dy);
}
```

### Combat System (CombatSystem.js)

Implements the warhead-versus-armor damage matrix and targeting priority.

```javascript
function calculateDamage(weapon, attacker, target) {
  const warhead = weapon.warhead;
  const armorType = target.stats.armor;
  const versesMultiplier = warhead.verses[armorType] / 100;
  
  let damage = weapon.damage * versesMultiplier;
  
  // Prone infantry modifier
  if (target.isProne && target.category === 'infantry') {
    damage *= 0.5;
  }
  
  // Area of effect falloff
  if (warhead.cellSpread > 0) {
    const distance = getDistance(attacker.position, target.position);
    const cellDistance = distance / CELL_SIZE;
    if (cellDistance > 0 && cellDistance <= warhead.cellSpread) {
      const falloff = 1 - (cellDistance / warhead.cellSpread) * (1 - warhead.percentAtMax);
      damage *= falloff;
    }
  }
  
  return Math.floor(damage);
}

function selectTarget(unit, potentialTargets) {
  // Filter targets based on weapon capabilities
  const validTargets = potentialTargets.filter(target => {
    const weapon = unit.weapons[0];
    if (!weapon) return false;
    
    const targetType = getTargetType(target);
    if (!weapon.targeting.canTarget.includes(targetType)) return false;
    if (weapon.targeting.cannotTarget.includes(targetType)) return false;
    
    const verses = weapon.warhead.verses[target.stats.armor];
    if (verses <= 1 && !unit.forceAttacking) return false; // Won't auto-acquire
    
    const distance = getDistance(unit.position, target.position);
    if (distance > weapon.range * CELL_SIZE) return false;
    
    return true;
  });
  
  if (validTargets.length === 0) return null;
  
  // Sort by priority
  validTargets.sort((a, b) => {
    // 1. Threatening units first
    const aThreating = a.targetId === unit.id ? 1 : 0;
    const bThreating = b.targetId === unit.id ? 1 : 0;
    if (aThreating !== bThreating) return bThreating - aThreating;
    
    // 2. Effectiveness (verses percentage)
    const weapon = unit.weapons[0];
    const aEffectiveness = weapon.warhead.verses[a.stats.armor];
    const bEffectiveness = weapon.warhead.verses[b.stats.armor];
    if (aEffectiveness !== bEffectiveness) return bEffectiveness - aEffectiveness;
    
    // 3. Distance (closer preferred)
    const aDist = getDistance(unit.position, a.position);
    const bDist = getDistance(unit.position, b.position);
    return aDist - bDist;
  });
  
  return validTargets[0];
}
```

### Production System (ProductionSystem.js)

Handles build queues, prerequisites, and timing.

```javascript
const BUILD_TIME_MULTIPLIER = 0.048; // Cost * 0.048 = seconds

function updateProduction(state, tick) {
  for (const [playerId, queues] of Object.entries(state.production)) {
    const player = state.players[playerId];
    const powerRatio = player.powerProduction / Math.max(1, player.powerConsumption);
    const speedModifier = powerRatio >= 1 ? 1.0 : 0.5; // Low power halves speed
    
    for (const [type, queue] of Object.entries(queues)) {
      if (queue.paused || queue.queue.length === 0) continue;
      
      const item = queue.queue[0];
      const definition = getDefinition(item.id, type);
      const buildTime = definition.cost * BUILD_TIME_MULTIPLIER;
      
      // Check for multiple production buildings
      const productionBuildings = countProductionBuildings(state, playerId, type);
      const multipleBonus = Math.min(productionBuildings, 2); // Cap at 2x speed
      
      const progressPerTick = (1 / (buildTime * TICK_RATE)) * speedModifier * multipleBonus;
      queue.progress += progressPerTick;
      
      if (queue.progress >= 1.0) {
        if (type === 'structure') {
          // Building ready for placement
          item.status = 'ready';
        } else {
          // Unit produced immediately
          spawnUnit(state, playerId, item.id, type);
          queue.queue.shift();
          queue.progress = 0;
        }
      }
    }
  }
}

function canBuild(state, playerId, itemId, type) {
  const definition = getDefinition(itemId, type);
  const player = state.players[playerId];
  
  // Check credits
  if (player.credits < definition.cost) return { canBuild: false, reason: 'Insufficient credits' };
  
  // Check prerequisites
  for (const prereq of definition.prerequisites) {
    const hasPrereq = Array.from(state.entities.buildings.values())
      .some(b => b.ownerId === playerId && b.id === prereq && b.health > 0);
    if (!hasPrereq) {
      return { canBuild: false, reason: `Requires ${prereq}` };
    }
  }
  
  // Check if production building exists
  const hasProductionBuilding = Array.from(state.entities.buildings.values())
    .some(b => b.ownerId === playerId && b.production?.type === type && b.health > 0);
  if (!hasProductionBuilding) {
    return { canBuild: false, reason: 'No production building' };
  }
  
  return { canBuild: true };
}
```

### Economy System (EconomySystem.js)

Manages credits, storage, and harvester behavior.

```javascript
const ORE_VALUE = 1;
const GEM_VALUE = 2;
const HARVESTER_CAPACITY = 700;
const HARVEST_RATE = 50; // Credits worth per tick when harvesting

function updateEconomy(state, tick) {
  // Update harvesters
  for (const [id, unit] of state.entities.units) {
    if (unit.id !== 'harvester') continue;
    
    switch (unit.harvesterState) {
      case 'idle':
        const nearestOre = findNearestResource(state, unit.position);
        if (nearestOre) {
          unit.harvesterState = 'movingToOre';
          issueMove(unit, nearestOre);
        }
        break;
        
      case 'movingToOre':
        if (hasArrived(unit)) {
          unit.harvesterState = 'harvesting';
        }
        break;
        
      case 'harvesting':
        const cell = getCellAt(state.map, unit.position);
        if (cell.ore > 0 && unit.cargo < HARVESTER_CAPACITY) {
          const harvestAmount = Math.min(HARVEST_RATE, cell.ore, HARVESTER_CAPACITY - unit.cargo);
          unit.cargo += harvestAmount * (cell.resourceType === 'gems' ? GEM_VALUE : ORE_VALUE);
          cell.ore -= harvestAmount / (cell.resourceType === 'gems' ? GEM_VALUE : ORE_VALUE);
        } else if (unit.cargo >= HARVESTER_CAPACITY || cell.ore === 0) {
          const refinery = findNearestRefinery(state, unit.position, unit.ownerId);
          if (refinery) {
            unit.harvesterState = 'returning';
            issueMove(unit, refinery.position);
          }
        }
        break;
        
      case 'returning':
        if (hasArrived(unit)) {
          unit.harvesterState = 'unloading';
        }
        break;
        
      case 'unloading':
        const player = state.players[unit.ownerId];
        const depositAmount = Math.min(unit.cargo, 50); // Deposit rate per tick
        const storageSpace = player.creditStorage - player.credits;
        const actualDeposit = Math.min(depositAmount, storageSpace);
        
        player.credits += actualDeposit;
        unit.cargo -= depositAmount; // Overflow is lost
        
        if (unit.cargo <= 0) {
          unit.harvesterState = 'idle';
        }
        break;
    }
  }
  
  // Ore regeneration
  if (tick % (TICK_RATE * 30) === 0) { // Every 30 seconds
    regenerateOre(state.map);
  }
}

function regenerateOre(map) {
  for (const resource of map.resources) {
    if (!resource.regenerates) continue;
    
    const cell = map.cells[resource.y][resource.x];
    if (cell.ore >= resource.density) {
      // Spread to adjacent cells
      for (const [nx, ny] of getAdjacentCells(resource.x, resource.y)) {
        const neighbor = map.cells[ny]?.[nx];
        if (neighbor && neighbor.type === 'clear' && neighbor.ore < resource.density) {
          neighbor.ore = Math.min(neighbor.ore + 1, resource.density);
          neighbor.resourceType = resource.type;
        }
      }
    } else {
      cell.ore = Math.min(cell.ore + 1, resource.density);
    }
  }
}
```

### Vision System (VisionSystem.js)

Implements fog of war with shroud and fog states.

```javascript
function updateVision(state, tick) {
  for (const [playerId, vision] of Object.entries(state.vision)) {
    const previousVisible = new Set(vision.visible);
    vision.visible.clear();
    
    // Units provide vision
    for (const [id, unit] of state.entities.units) {
      if (unit.ownerId !== parseInt(playerId)) continue;
      revealArea(vision, unit.position, unit.stats.sightRange, state.map);
    }
    
    // Buildings provide vision
    for (const [id, building] of state.entities.buildings) {
      if (building.ownerId !== parseInt(playerId)) continue;
      const centerX = building.x + Math.floor(building.footprint.width / 2);
      const centerY = building.y + Math.floor(building.footprint.height / 2);
      revealArea(vision, { x: centerX, y: centerY }, building.stats.sightRange, state.map);
    }
    
    // Update explored (never goes back to shroud)
    for (const cell of vision.visible) {
      vision.explored.add(cell);
    }
  }
}

function revealArea(vision, center, range, map) {
  const cellX = Math.floor(center.x / CELL_SIZE);
  const cellY = Math.floor(center.y / CELL_SIZE);
  
  for (let dy = -range; dy <= range; dy++) {
    for (let dx = -range; dx <= range; dx++) {
      const distance = Math.sqrt(dx * dx + dy * dy);
      if (distance > range) continue;
      
      const x = cellX + dx;
      const y = cellY + dy;
      
      if (x < 0 || x >= map.width || y < 0 || y >= map.height) continue;
      
      // Line of sight check for terrain blocking
      if (hasLineOfSight(cellX, cellY, x, y, map)) {
        vision.visible.add(`${x},${y}`);
      }
    }
  }
}

function hasLineOfSight(x1, y1, x2, y2, map) {
  const dx = Math.abs(x2 - x1);
  const dy = Math.abs(y2 - y1);
  const sx = x1 < x2 ? 1 : -1;
  const sy = y1 < y2 ? 1 : -1;
  let err = dx - dy;
  let x = x1;
  let y = y1;
  
  while (x !== x2 || y !== y2) {
    const cell = map.cells[y]?.[x];
    if (cell && cell.blocksVision) return false;
    
    const e2 = 2 * err;
    if (e2 > -dy) { err -= dy; x += sx; }
    if (e2 < dx) { err += dx; y += sy; }
  }
  
  return true;
}

function getVisibility(state, playerId, x, y) {
  const vision = state.vision[playerId];
  const key = `${x},${y}`;
  
  if (vision.visible.has(key)) return 'visible';
  if (vision.explored.has(key)) return 'fog';
  return 'shroud';
}
```

### Power System (PowerSystem.js)

```javascript
function updatePower(state, tick) {
  for (const player of state.players) {
    let production = 0;
    let consumption = 0;
    
    for (const [id, building] of state.entities.buildings) {
      if (building.ownerId !== player.id) continue;
      if (building.health <= 0) continue;
      
      const definition = getBuildingDefinition(building.id);
      
      // Damaged buildings produce proportionally less power
      const healthRatio = building.health / definition.stats.health;
      
      if (definition.power.production > 0) {
        production += definition.power.production * healthRatio;
      }
      if (definition.power.consumption > 0) {
        consumption += definition.power.consumption;
      }
    }
    
    player.powerProduction = Math.floor(production);
    player.powerConsumption = consumption;
    
    // Apply low power effects
    const lowPower = player.powerConsumption > player.powerProduction;
    
    for (const [id, building] of state.entities.buildings) {
      if (building.ownerId !== player.id) continue;
      
      const definition = getBuildingDefinition(building.id);
      
      // Disable powered defenses when low power
      if (definition.requiresPower) {
        building.disabled = lowPower;
      }
      
      // Disable radar
      if (building.id === 'radarDome' || building.id === 'radarTower') {
        building.radarActive = !lowPower;
      }
    }
  }
}
```

## Component Specifications

### GameCanvas.jsx

```jsx
import { Canvas } from '@react-three/fiber';
import { OrthographicCamera } from '@react-three/drei';

function GameCanvas() {
  const { camera } = useGameStore();
  
  return (
    <Canvas
      orthographic
      camera={{
        zoom: 50 * camera.zoom,
        position: [camera.x + 10, 10, camera.y + 10],
        near: 0.1,
        far: 1000,
      }}
      gl={{ antialias: true }}
    >
      <ambientLight intensity={0.6} />
      <directionalLight position={[10, 20, 10]} intensity={0.8} />
      
      <IsometricCamera />
      <TerrainGrid />
      <ResourceFields />
      <Buildings />
      <Units />
      <Projectiles />
      <FogOfWar />
      <SelectionBox />
      <BuildingPlacementPreview />
    </Canvas>
  );
}
```

### TerrainCell.jsx

Uses vertex colors for wireframe-style terrain without textures.

```jsx
import { useMemo } from 'react';
import * as THREE from 'three';

const TERRAIN_COLORS = {
  clear: '#4A7C59',
  road: '#8B7355',
  rough: '#6B5344',
  water: '#2E5A88',
  ore: '#D4A574',
  gems: '#7B68EE',
  cliff: '#5C5C5C',
};

function TerrainCell({ x, y, type, elevation, ore }) {
  const geometry = useMemo(() => {
    const geo = new THREE.PlaneGeometry(1, 1, 1, 1);
    
    // Apply elevation to vertices
    const positions = geo.attributes.position.array;
    for (let i = 0; i < positions.length; i += 3) {
      positions[i + 2] = elevation * 0.2; // Z becomes Y in isometric
    }
    
    // Apply vertex colors
    const colors = [];
    const baseColor = new THREE.Color(TERRAIN_COLORS[type]);
    
    // Darken edges for wireframe effect
    for (let i = 0; i < 4; i++) {
      const edgeFactor = 0.7 + Math.random() * 0.3; // Slight variation
      colors.push(
        baseColor.r * edgeFactor,
        baseColor.g * edgeFactor,
        baseColor.b * edgeFactor
      );
    }
    
    geo.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));
    geo.computeVertexNormals();
    
    return geo;
  }, [type, elevation]);
  
  // Convert to isometric coordinates
  const isoX = (x - y) * 0.5;
  const isoY = elevation * 0.2;
  const isoZ = (x + y) * 0.5;
  
  return (
    <mesh
      geometry={geometry}
      position={[isoX, isoY, isoZ]}
      rotation={[-Math.PI / 2, 0, Math.PI / 4]}
    >
      <meshBasicMaterial
        vertexColors
        wireframe={false}
        side={THREE.DoubleSide}
      />
    </mesh>
  );
}
```

### UnitRenderer.jsx

Wireframe 3D unit with faction-colored vertex shading.

```jsx
import { useRef, useMemo } from 'react';
import { useFrame } from '@react-three/fiber';
import * as THREE from 'three';

function UnitRenderer({ unit, definition, selected }) {
  const meshRef = useRef();
  const { players } = useGameStore();
  const player = players[unit.ownerId];
  
  const geometry = useMemo(() => {
    // Generate unit geometry based on definition.model.meshId
    const geo = createUnitGeometry(definition.model.meshId);
    
    // Apply faction colors to vertices
    const colors = [];
    const primaryColor = new THREE.Color(player.color);
    const positions = geo.attributes.position.array;
    
    for (let i = 0; i < positions.length / 3; i++) {
      const y = positions[i * 3 + 1];
      // Top vertices get primary color, bottom gets darker shade
      const colorFactor = 0.5 + (y / 2) * 0.5;
      colors.push(
        primaryColor.r * colorFactor,
        primaryColor.g * colorFactor,
        primaryColor.b * colorFactor
      );
    }
    
    geo.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));
    return geo;
  }, [definition, player.color]);
  
  // Convert world position to isometric
  const isoX = (unit.position.x - unit.position.y) * 0.5;
  const isoY = unit.position.z || 0;
  const isoZ = (unit.position.x + unit.position.y) * 0.5;
  
  useFrame(() => {
    if (meshRef.current) {
      meshRef.current.rotation.y = unit.facing * (Math.PI / 180);
    }
  });
  
  return (
    <group position={[isoX, isoY, isoZ]}>
      <mesh ref={meshRef} geometry={geometry}>
        <meshBasicMaterial vertexColors />
      </mesh>
      
      {/* Turret for vehicles */}
      {definition.weapons[0]?.turret && (
        <UnitTurret
          unit={unit}
          weapon={definition.weapons[0]}
          playerColor={player.color}
        />
      )}
      
      {/* Selection ring */}
      {selected && <SelectionRing radius={0.5} color={player.color} />}
      
      {/* Health bar */}
      <HealthBar
        current={unit.health}
        max={definition.stats.health}
        position={[0, 1.2, 0]}
      />
    </group>
  );
}

function createUnitGeometry(meshId) {
  // Simplified wireframe geometries for each unit type
  const geometries = {
    'infantry_rifleman': () => {
      const geo = new THREE.BoxGeometry(0.3, 0.6, 0.3);
      return geo;
    },
    'tank_light': () => {
      const geo = new THREE.BoxGeometry(0.8, 0.3, 1.0);
      return geo;
    },
    'tank_medium': () => {
      const geo = new THREE.BoxGeometry(0.9, 0.35, 1.1);
      return geo;
    },
    'tank_heavy': () => {
      const geo = new THREE.BoxGeometry(1.0, 0.4, 1.3);
      return geo;
    },
    'harvester': () => {
      const geo = new THREE.BoxGeometry(1.0, 0.5, 1.2);
      return geo;
    },
  };
  
  return geometries[meshId]?.() || new THREE.BoxGeometry(0.5, 0.5, 0.5);
}
```

### Sidebar.jsx

```jsx
function Sidebar() {
  const { players, selection, production } = useGameStore();
  const currentPlayer = players[0]; // Human player
  const [activeTab, setActiveTab] = useState('structures');
  
  return (
    <div className="fixed right-0 top-0 h-full w-64 bg-gray-900 flex flex-col">
      {/* Credits Display */}
      <CreditsDisplay credits={currentPlayer.credits} />
      
      {/* Power Bar */}
      <PowerBar
        production={currentPlayer.powerProduction}
        consumption={currentPlayer.powerConsumption}
      />
      
      {/* Minimap */}
      <Minimap />
      
      {/* Repair/Sell Buttons */}
      <div className="flex gap-2 p-2">
        <RepairButton />
        <SellButton />
      </div>
      
      {/* Build Tabs */}
      <BuildTabs activeTab={activeTab} onTabChange={setActiveTab} />
      
      {/* Build Grid */}
      <BuildGrid
        tab={activeTab}
        queue={production[0][activeTab]}
        playerId={0}
      />
      
      {/* Selected Unit Info (if units selected) */}
      {selection.units.length > 0 && (
        <UnitInfoPanel units={selection.units} />
      )}
      
      {/* Command Panel */}
      {selection.units.length > 0 && (
        <CommandPanel units={selection.units} />
      )}
    </div>
  );
}
```

### BuildButton.jsx

```jsx
function BuildButton({ item, queue, canBuild, onClick }) {
  const definition = getDefinition(item.id, item.type);
  const isBuilding = queue.queue[0]?.id === item.id;
  const progress = isBuilding ? queue.progress : 0;
  const status = getButtonStatus(item, queue, canBuild);
  
  return (
    <button
      onClick={() => canBuild.canBuild && onClick(item.id)}
      className={`
        relative w-12 h-12 border-2 
        ${status === 'available' ? 'border-green-500 hover:border-green-400' : ''}
        ${status === 'building' ? 'border-yellow-500' : ''}
        ${status === 'ready' ? 'border-blue-500 animate-pulse' : ''}
        ${status === 'unavailable' ? 'border-gray-600 opacity-50' : ''}
      `}
      disabled={!canBuild.canBuild}
      title={canBuild.canBuild ? definition.name : canBuild.reason}
    >
      {/* Unit/Building Icon (SVG) */}
      <UnitIcon unitId={item.id} />
      
      {/* Progress Bar Overlay */}
      {isBuilding && (
        <div
          className="absolute bottom-0 left-0 right-0 bg-yellow-500 opacity-50"
          style={{ height: `${progress * 100}%` }}
        />
      )}
      
      {/* Cost */}
      <span className="absolute bottom-0 right-0 text-xs text-white bg-black px-1">
        ${definition.cost}
      </span>
      
      {/* Lock Icon for Unavailable */}
      {!canBuild.canBuild && (
        <div className="absolute inset-0 flex items-center justify-center">
          <LockIcon className="w-6 h-6 text-gray-400" />
        </div>
      )}
    </button>
  );
}
```

### Minimap.jsx

```jsx
function Minimap() {
  const canvasRef = useRef();
  const { map, entities, players, vision, camera } = useGameStore();
  const currentPlayer = players[0];
  
  const hasRadar = useMemo(() => {
    return Array.from(entities.buildings.values()).some(
      b => b.ownerId === 0 && 
           (b.id === 'radarDome' || b.id === 'radarTower') && 
           b.radarActive
    );
  }, [entities.buildings]);
  
  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    // Clear
    ctx.fillStyle = '#1a1a1a';
    ctx.fillRect(0, 0, 128, 128);
    
    if (!hasRadar) {
      // Show faction logo instead
      drawFactionLogo(ctx, currentPlayer.faction);
      return;
    }
    
    const scaleX = 128 / map.width;
    const scaleY = 128 / map.height;
    
    // Draw terrain
    for (let y = 0; y < map.height; y++) {
      for (let x = 0; x < map.width; x++) {
        const visibility = getVisibility(vision[0], x, y);
        if (visibility === 'shroud') continue;
        
        const cell = map.cells[y][x];
        ctx.fillStyle = visibility === 'fog' 
          ? darken(MINIMAP_COLORS[cell.type], 0.5)
          : MINIMAP_COLORS[cell.type];
        ctx.fillRect(x * scaleX, y * scaleY, scaleX, scaleY);
      }
    }
    
    // Draw buildings
    for (const [id, building] of entities.buildings) {
      const visibility = getVisibility(vision[0], building.x, building.y);
      if (visibility === 'shroud') continue;
      if (visibility === 'fog' && building.ownerId !== 0) continue;
      
      const player = players[building.ownerId];
      ctx.fillStyle = player?.color || '#808080';
      ctx.fillRect(
        building.x * scaleX,
        building.y * scaleY,
        building.footprint.width * scaleX,
        building.footprint.height * scaleY
      );
    }
    
    // Draw units
    for (const [id, unit] of entities.units) {
      const cellX = Math.floor(unit.position.x / CELL_SIZE);
      const cellY = Math.floor(unit.position.y / CELL_SIZE);
      const visibility = getVisibility(vision[0], cellX, cellY);
      
      if (visibility !== 'visible' && unit.ownerId !== 0) continue;
      
      const player = players[unit.ownerId];
      ctx.fillStyle = player?.color || '#808080';
      ctx.fillRect(cellX * scaleX - 1, cellY * scaleY - 1, 3, 3);
    }
    
    // Draw camera viewport
    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 1;
    ctx.strokeRect(
      camera.x * scaleX,
      camera.y * scaleY,
      (camera.viewWidth / CELL_SIZE) * scaleX,
      (camera.viewHeight / CELL_SIZE) * scaleY
    );
  }, [map, entities, vision, camera, hasRadar]);
  
  const handleClick = (e) => {
    if (!hasRadar) return;
    
    const rect = canvasRef.current.getBoundingClientRect();
    const x = (e.clientX - rect.left) / 128 * map.width;
    const y = (e.clientY - rect.top) / 128 * map.height;
    
    useGameStore.getState().actions.moveCamera(x, y);
  };
  
  return (
    <canvas
      ref={canvasRef}
      width={128}
      height={128}
      onClick={handleClick}
      className="border border-gray-600 cursor-pointer"
    />
  );
}
```

## Level Editor Specification

### LevelEditor.jsx

```jsx
function LevelEditor() {
  const {
    map,
    selectedTool,
    selectedBrush,
    selectedEntity,
    triggers,
  } = useEditorStore();
  
  return (
    <div className="flex h-screen">
      {/* Canvas Area */}
      <div className="flex-1 relative">
        <Canvas orthographic>
          <EditorCamera />
          <EditorTerrainGrid />
          <EditorEntities />
          <EditorGrid showCoordinates />
          <EditorCursor tool={selectedTool} brush={selectedBrush} />
        </Canvas>
        
        {/* Coordinate Display */}
        <div className="absolute bottom-2 left-2 bg-black/50 px-2 py-1 text-white">
          {hoveredCell.x}, {hoveredCell.y}
        </div>
      </div>
      
      {/* Toolbar */}
      <EditorToolbar />
      
      {/* Property Panel */}
      <PropertyPanel />
    </div>
  );
}
```

### EditorToolbar.jsx

```jsx
const TOOLS = [
  { id: 'select', icon: CursorIcon, label: 'Select' },
  { id: 'terrain', icon: MountainIcon, label: 'Terrain' },
  { id: 'resource', icon: GemIcon, label: 'Resources' },
  { id: 'building', icon: BuildingIcon, label: 'Buildings' },
  { id: 'unit', icon: TankIcon, label: 'Units' },
  { id: 'startPos', icon: FlagIcon, label: 'Start Positions' },
  { id: 'trigger', icon: LightningIcon, label: 'Triggers' },
];

const TERRAIN_BRUSHES = [
  { id: 'clear', color: '#4A7C59', label: 'Clear' },
  { id: 'road', color: '#8B7355', label: 'Road' },
  { id: 'rough', color: '#6B5344', label: 'Rough' },
  { id: 'water', color: '#2E5A88', label: 'Water' },
  { id: 'cliff', color: '#5C5C5C', label: 'Cliff' },
];

function EditorToolbar() {
  const { selectedTool, setTool, selectedBrush, setBrush, brushSize, setBrushSize } = useEditorStore();
  
  return (
    <div className="w-16 bg-gray-800 flex flex-col items-center py-2 gap-2">
      {/* Tool Selection */}
      {TOOLS.map(tool => (
        <button
          key={tool.id}
          onClick={() => setTool(tool.id)}
          className={`
            w-12 h-12 flex items-center justify-center rounded
            ${selectedTool === tool.id ? 'bg-blue-600' : 'bg-gray-700 hover:bg-gray-600'}
          `}
          title={tool.label}
        >
          <tool.icon className="w-6 h-6 text-white" />
        </button>
      ))}
      
      <div className="border-t border-gray-600 w-full my-2" />
      
      {/* Terrain Brushes (shown when terrain tool selected) */}
      {selectedTool === 'terrain' && TERRAIN_BRUSHES.map(brush => (
        <button
          key={brush.id}
          onClick={() => setBrush(brush.id)}
          className={`
            w-10 h-10 rounded border-2
            ${selectedBrush === brush.id ? 'border-white' : 'border-transparent'}
          `}
          style={{ backgroundColor: brush.color }}
          title={brush.label}
        />
      ))}
      
      {/* Brush Size */}
      {selectedTool === 'terrain' && (
        <div className="flex flex-col items-center gap-1">
          <label className="text-xs text-gray-400">Size</label>
          <input
            type="range"
            min={1}
            max={5}
            value={brushSize}
            onChange={(e) => setBrushSize(parseInt(e.target.value))}
            className="w-12"
          />
          <span className="text-xs text-white">{brushSize}</span>
        </div>
      )}
      
      <div className="flex-1" />
      
      {/* File Operations */}
      <button onClick={handleNew} className="w-12 h-12 bg-gray-700 hover:bg-gray-600 rounded" title="New Map">
        <FileIcon className="w-6 h-6 text-white mx-auto" />
      </button>
      <button onClick={handleSave} className="w-12 h-12 bg-gray-700 hover:bg-gray-600 rounded" title="Save Map">
        <SaveIcon className="w-6 h-6 text-white mx-auto" />
      </button>
      <button onClick={handleLoad} className="w-12 h-12 bg-gray-700 hover:bg-gray-600 rounded" title="Load Map">
        <FolderIcon className="w-6 h-6 text-white mx-auto" />
      </button>
      <button onClick={handleExport} className="w-12 h-12 bg-green-700 hover:bg-green-600 rounded" title="Export JSON">
        <DownloadIcon className="w-6 h-6 text-white mx-auto" />
      </button>
    </div>
  );
}
```

### Map Export Format

The editor exports maps as self-contained JSON files:

```javascript
function exportMap(editorState) {
  const map = {
    meta: {
      name: editorState.mapName,
      author: editorState.author,
      version: '1.0.0',
      description: editorState.description,
      players: editorState.startPositions.length,
      size: { width: editorState.width, height: editorState.height },
      createdAt: new Date().toISOString(),
    },
    terrain: {
      cells: editorState.cells.flatMap((row, y) =>
        row.map((cell, x) => ({
          x, y,
          type: cell.type,
          elevation: cell.elevation,
          ore: cell.ore,
          passability: getPassability(cell.type),
        }))
      ),
      overlays: editorState.overlays,
    },
    resources: editorState.resources,
    startPositions: editorState.startPositions,
    preplacedEntities: editorState.entities,
    triggers: editorState.triggers,
    options: editorState.options,
  };
  
  return JSON.stringify(map, null, 2);
}
```

### TriggerEditor.jsx

```jsx
const TRIGGER_CONDITIONS = [
  { id: 'timeElapsed', label: 'Time Elapsed', params: ['seconds'] },
  { id: 'unitDestroyed', label: 'Unit Destroyed', params: ['unitId'] },
  { id: 'buildingCaptured', label: 'Building Captured', params: ['buildingId', 'byPlayer'] },
  { id: 'areaEntered', label: 'Area Entered', params: ['area', 'byPlayer'] },
  { id: 'creditsReached', label: 'Credits Reached', params: ['player', 'amount'] },
];

const TRIGGER_ACTIONS = [
  { id: 'spawnUnits', label: 'Spawn Units', params: ['units', 'at', 'owner'] },
  { id: 'revealArea', label: 'Reveal Area', params: ['area', 'forPlayer'] },
  { id: 'playSound', label: 'Play Sound', params: ['soundId'] },
  { id: 'showMessage', label: 'Show Message', params: ['text', 'duration'] },
  { id: 'setObjective', label: 'Set Objective', params: ['text', 'type'] },
  { id: 'victory', label: 'Victory', params: ['player'] },
  { id: 'defeat', label: 'Defeat', params: ['player'] },
];

function TriggerEditor({ trigger, onChange }) {
  return (
    <div className="bg-gray-800 p-4 rounded">
      <input
        value={trigger.id}
        onChange={(e) => onChange({ ...trigger, id: e.target.value })}
        placeholder="Trigger ID"
        className="w-full bg-gray-700 text-white px-2 py-1 rounded mb-2"
      />
      
      {/* Condition */}
      <div className="mb-4">
        <label className="text-sm text-gray-400">Condition</label>
        <select
          value={trigger.condition.type}
          onChange={(e) => onChange({
            ...trigger,
            condition: { type: e.target.value }
          })}
          className="w-full bg-gray-700 text-white px-2 py-1 rounded"
        >
          {TRIGGER_CONDITIONS.map(c => (
            <option key={c.id} value={c.id}>{c.label}</option>
          ))}
        </select>
        
        {/* Condition Parameters */}
        <ConditionParams
          condition={trigger.condition}
          onChange={(condition) => onChange({ ...trigger, condition })}
        />
      </div>
      
      {/* Action */}
      <div>
        <label className="text-sm text-gray-400">Action</label>
        <select
          value={trigger.action.type}
          onChange={(e) => onChange({
            ...trigger,
            action: { type: e.target.value }
          })}
          className="w-full bg-gray-700 text-white px-2 py-1 rounded"
        >
          {TRIGGER_ACTIONS.map(a => (
            <option key={a.id} value={a.id}>{a.label}</option>
          ))}
        </select>
        
        {/* Action Parameters */}
        <ActionParams
          action={trigger.action}
          onChange={(action) => onChange({ ...trigger, action })}
        />
      </div>
    </div>
  );
}
```

## Keyboard Controls

| Key | Action |
|-----|--------|
| **Selection** | |
| Left Click | Select unit/building |
| Shift + Click | Add to selection |
| Ctrl + Click | Remove from selection |
| Drag | Box select |
| Double Click | Select all same type on screen |
| 0-9 | Select control group |
| Ctrl + 0-9 | Assign control group |
| **Camera** | |
| Arrow Keys / WASD | Pan camera |
| Mouse Edge | Pan camera |
| Scroll Wheel | Zoom |
| Spacebar | Center on selection |
| Home | Center on Construction Yard |
| **Commands** | |
| Right Click | Move / Attack / Harvest |
| A | Attack move |
| S | Stop |
| G | Guard |
| D | Deploy (MCV) |
| Ctrl + Right Click | Force attack |
| **Production** | |
| Q | Structures tab |
| W | Defense tab |
| E | Infantry tab |
| R | Vehicles tab |
| Escape | Cancel placement |
| **Game** | |
| F1-F4 | Set camera bookmark |
| Shift + F1-F4 | Jump to bookmark |
| P | Pause |
| + / - | Game speed |

## Multiplayer Considerations

While initial implementation is single-player, the architecture supports future multiplayer through:

1. **Deterministic Simulation**: Fixed tick rate with seeded random ensures identical simulation across clients
2. **Command-Based Input**: All player actions are commands that can be serialized and transmitted
3. **State Synchronization**: Game state can be serialized to JSON for sync verification
4. **Lockstep Ready**: Game loop structure supports lockstep networking where clients wait for all inputs before advancing

```javascript
// Command structure for network serialization
const command = {
  tick: 1234,
  playerId: 0,
  type: 'move',
  unitIds: [1, 2, 3],
  target: { x: 100, y: 200 },
  queued: false,
};

// Deterministic random using seed
function seededRandom(seed) {
  const x = Math.sin(seed++) * 10000;
  return x - Math.floor(x);
}
```

## Testing Scenarios

### Unit Tests

1. **Pathfinding**: Verify A* finds optimal paths around obstacles
2. **Combat**: Confirm damage calculations match warhead/armor matrix
3. **Production**: Validate build times and prerequisite checking
4. **Economy**: Test harvester loops and storage overflow
5. **Vision**: Ensure fog of war updates correctly with unit movement

### Integration Tests

1. **Full Game Loop**: Run 1000 ticks and verify state consistency
2. **Save/Load**: Save game state, reload, continue without errors
3. **Map Loading**: Load all included maps without errors
4. **Edge Cases**: Zero credits, destroyed Construction Yard, all units killed

### Visual Tests

1. **Isometric Rendering**: Verify correct depth sorting
2. **Selection Box**: Confirm accurate unit selection
3. **Fog of War**: Visual inspection of shroud/fog transitions
4. **UI Responsiveness**: All buttons clickable and responsive

## Performance Goals

- **60 FPS** rendering with 200 units on screen
- **15 TPS** game simulation (fixed)
- **<100ms** pathfinding for cross-map paths
- **<16ms** per render frame
- **<50MB** initial memory footprint
- Map sizes up to **128x128** cells

## Accessibility Requirements

1. **Keyboard Navigation**: All UI elements accessible via keyboard
2. **Color Blind Support**: Faction colors distinguishable; optional patterns
3. **Screen Reader**: ARIA labels on all interactive elements
4. **Pause Anytime**: Full pause with no time pressure
5. **Adjustable Speed**: 0.5x to 2x game speed
6. **Large UI Option**: Scaled sidebar for visibility

## File Manifest

When implementation is complete, the project will contain:

```
/
├── README.md (this file)
├── package.json
├── vite.config.js
├── tailwind.config.js
├── index.html
├── public/
│   └── sounds/ (placeholder for audio)
└── src/
    ├── App.jsx
    ├── main.jsx
    ├── components/ (as specified above)
    ├── systems/ (as specified above)
    ├── stores/ (as specified above)
    ├── data/ (JSON definitions)
    ├── hooks/ (as specified above)
    └── utils/ (as specified above)
```

## Version History

- **1.0.0** - Initial TINS specification based on Red Alert research study
