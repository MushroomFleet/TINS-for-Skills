# REACT-WARS

## Description

REACT-WARS is a real-time tactical squad combat game built entirely in React/TypeScript, directly inspired by Bullfrog's 1996 classic *Syndicate Wars*. Players command a squad of four cyborg agents through isometric 3D cityscapes, completing missions involving assassination, extraction, sabotage, and territory control.

The game features low-poly wireframe 3D models rendered with flat SVGA-style shading (vertex colors, no textures), creating a distinctive cyberpunk aesthetic while remaining performant in the browser. All game systems—squad control, the IPA drug system, cybernetic upgrades, research trees, destructible environments, and the iconic Persuadertron—are faithfully recreated using a component-based architecture with strict separation of concerns.

The architecture prioritizes modularity: no single file exceeds 200 lines, game logic is separated from rendering, and state management flows predictably through a centralized store. This enables easy extension, testing, and maintenance.

---

## Functionality

### Core Game Loop

1. **Strategic Layer (World Map)**
   - View global territory control between EuroCorp and Church of the New Epoch
   - Select missions from available territories
   - Manage research queue and agent loadouts
   - Review mission briefings and objectives

2. **Tactical Layer (Mission)**
   - Real-time squad control in isometric 3D environment
   - Complete primary and secondary objectives
   - Extract agents to end mission
   - Capture technology and personnel for research

3. **Progression Layer**
   - Research captured tech to unlock weapons and upgrades
   - Install cybernetic modifications on agents
   - Expand territory control through successful missions

### Squad Control System

**Selection**
- Click individual agent to select
- Ctrl+Click to add/remove from selection
- Drag-box to select multiple agents
- Number keys 1-4 select individual agents
- Key 5 selects all agents

**Movement**
- Right-click ground to move selected agents
- Agents pathfind around obstacles automatically
- Hold Shift+Right-click to queue waypoints
- Agents maintain formation during movement

**Combat**
- Right-click enemy to attack
- Agents auto-acquire targets within perception range
- Hold position with 'H' key
- Aggressive stance with 'A' key (auto-engage)

### IPA (Intelligence, Perception, Adrenaline) System

Each agent has three drug levels adjustable in real-time (0-100%):

| Drug | Low Setting | High Setting |
|------|-------------|--------------|
| **Intelligence** | Fast reactions, poor accuracy | Slow reactions, excellent accuracy |
| **Perception** | Narrow vision cone, miss threats | Wide vision, spot cloaked units |
| **Adrenaline** | Slow movement, precise control | Fast movement, erratic behavior |

**UI Controls:**
- Three vertical sliders per agent in the Agent Panel
- Drag to adjust, double-click to reset to 50%
- Color coding: Blue (INT), Yellow (PER), Red (ADR)
- Preset buttons: "Combat" (30/70/60), "Stealth" (70/90/20), "Assault" (20/40/100)

### Weapons System

**Weapon Categories:**

```
SMALL ARMS
├── Pistol         - Accurate, low damage, unlimited ammo
├── Uzi            - Fast fire rate, medium accuracy
└── Shotgun        - High close damage, spread pattern

HEAVY WEAPONS
├── Minigun        - Extreme fire rate, slow movement while firing
├── Flamer         - Cone damage, ignites environment
└── Electron Mace  - Melee, stun effect, shield bypass

EXPLOSIVES
├── Launcher       - Projectile, area damage, destroys structures
├── High Explosive - Throwable, timed detonation
└── Nuclear Grenade - Massive radius, radiation DOT

ENERGY WEAPONS
├── Laser          - Instant hit, high accuracy, penetrates targets
├── Pulse Laser    - Burst fire, bonus vs cybernetics
└── Graviton Gun   - Ragdoll physics, environmental kills

SPECIAL
├── Psycho Gas     - Area denial, enemies attack each other
└── Satellite Rain - Orbital strike, 5 second delay, massive damage

EQUIPMENT
├── Medikit        - Heal 50 HP, 3 uses
├── Energy Shield  - Absorb 200 damage, 10 second duration
├── Cloak          - Invisibility, breaks on attack
└── Persuadertron  - Mind control civilians/enemies
```

**Weapon Slots:**
- Each agent has 4 weapon slots
- Switch weapons with Q/E or mouse wheel
- Ammo displayed per weapon in HUD

### Persuadertron Mechanics

The Persuadertron converts civilians and enemy agents to your control:

**Conversion Formula:**
```
ConversionPower = (NumAgents × AgentLevel) + CyberwareBonus
TargetResistance = TargetType × TargetCyberLevel

Success when: ConversionPower > TargetResistance
```

**Target Types:**
| Target | Base Resistance |
|--------|-----------------|
| Civilian | 10 |
| Police | 30 |
| Enemy Agent | 50 + (25 × CyberLevel) |

**Converted Units:**
- Follow selected agent automatically
- Can be armed with dropped weapons
- Die permanently (not extracted)
- Maximum 20 converted units at once

### Cybernetic Upgrades

**Body Parts (per agent):**
```
LEGS (v1/v2/v3)
├── v1: +10% move speed
├── v2: +25% move speed, jump ability
└── v3: +40% move speed, double jump

ARMS (v1/v2/v3)
├── v1: +10% accuracy, +10% melee damage
├── v2: +25% accuracy, dual wield small arms
└── v3: +40% accuracy, dual wield heavy weapons

TORSO (v1/v2/v3)
├── v1: +50 max HP
├── v2: +100 max HP, damage reduction 10%
└── v3: +150 max HP, damage reduction 25%

BRAIN (v1/v2/v3)
├── v1: +20% IPA effectiveness
├── v2: +40% IPA effectiveness, auto-dodge 10%
└── v3: +60% IPA effectiveness, auto-dodge 25%

EYES (v1/v2/v3)
├── v1: +20% perception range
├── v2: +40% perception range, see cloaked
└── v3: +60% perception range, thermal vision (through walls)
```

### Research System

**Research Flow:**
1. Capture items during missions (weapons, cyberware, data)
2. Items appear in Research Queue
3. Allocate scientists (1-10) to research
4. Research completes based on complexity and scientists
5. Unlocked items available for purchase/installation

**Research Time Formula:**
```
Days = BaseComplexity / (Scientists × LabLevel)
```

**Tech Tree Branches:**
- Weapons (captured enemy weapons)
- Cyberware (captured enemy implants)
- Vehicles (captured vehicle hulks)
- Special (mission data, Church/Corp secrets)

### Destructible Environments

**Structure Types:**
```
LIGHT (HP: 100)
├── Fences, signs, furniture
└── Destroyed by: Any weapon

MEDIUM (HP: 500)
├── Walls, doors, small structures
└── Destroyed by: Heavy weapons, explosives

HEAVY (HP: 2000)
├── Buildings, bridges, towers
└── Destroyed by: Explosives, Satellite Rain

INDESTRUCTIBLE
├── Ground, mission-critical structures
└── Visual damage only
```

**Destruction Effects:**
- Debris spawns as physics objects
- Fire spreads to adjacent flammable structures
- Civilians flee destruction
- Pathfinding updates dynamically

### Vehicles

**Vehicle Types:**
```
CAR
├── Speed: Fast | Armor: Light | Weapon: None
└── Capacity: 4 agents

APC  
├── Speed: Medium | Armor: Heavy | Weapon: Minigun
└── Capacity: 4 agents + 8 converts

TANK
├── Speed: Slow | Armor: Very Heavy | Weapon: Cannon
└── Capacity: 2 agents
```

**Vehicle Controls:**
- Enter/Exit with 'F' key when adjacent
- Drive with WASD or click-to-move
- Fire vehicle weapon with Left Click
- Vehicles can run over infantry

### Mission Types

1. **Assassination** - Eliminate specific target(s)
2. **Extraction** - Rescue/capture VIP and extract
3. **Sabotage** - Destroy target structure(s)
4. **Territory** - Eliminate all enemy presence
5. **Research** - Capture specific technology
6. **Defense** - Protect location for time limit

### Campaign Structure

**EuroCorp Campaign:**
- Maintain corporate control
- Suppress Church uprising
- Tech focus: Heavy weapons, vehicle upgrades
- 25 missions across 5 regions

**Church of the New Epoch Campaign:**
- Liberate minds from CHIP control
- Overthrow corporate dominance  
- Tech focus: Stealth, psychic weapons
- 25 missions across 5 regions

---

## Technical Implementation

### Architecture Overview

```
src/
├── components/           # React UI components
│   ├── hud/             # In-game HUD elements
│   ├── menus/           # Menu screens
│   ├── panels/          # Side panels (agent, research)
│   └── common/          # Shared UI components
├── engine/              # Game engine core
│   ├── renderer/        # Three.js rendering
│   ├── physics/         # Collision & physics
│   ├── pathfinding/     # A* navigation
│   └── audio/           # Sound system
├── systems/             # Game logic systems
│   ├── combat/          # Damage, weapons, projectiles
│   ├── ai/              # Enemy AI, behaviors
│   ├── squad/           # Agent control, IPA
│   └── world/           # Terrain, structures, vehicles
├── models/              # 3D model definitions
│   ├── agents/          # Character models
│   ├── weapons/         # Weapon models
│   ├── vehicles/        # Vehicle models
│   ├── structures/      # Building models
│   └── effects/         # Particle systems
├── state/               # State management
│   ├── store.ts         # Central game store
│   ├── slices/          # State slices
│   └── selectors.ts     # Derived state
├── data/                # Static game data
│   ├── weapons.ts       # Weapon definitions
│   ├── cyberware.ts     # Upgrade definitions
│   ├── missions.ts      # Mission configurations
│   └── research.ts      # Tech tree data
├── hooks/               # Custom React hooks
├── utils/               # Utility functions
└── types/               # TypeScript definitions
```

### Component Size Rules

- **Maximum 200 lines per file**
- **Single responsibility per component**
- **Extract hooks for complex logic**
- **Separate render logic from game logic**

### State Management

Using Zustand for lightweight, performant state:

```typescript
// types/GameState.ts
interface GameState {
  // Global
  phase: 'menu' | 'strategic' | 'tactical' | 'paused';
  campaign: 'eurocorp' | 'church';
  
  // Strategic
  territories: Territory[];
  research: ResearchState;
  funds: number;
  
  // Tactical
  agents: Agent[];
  enemies: Enemy[];
  civilians: Civilian[];
  structures: Structure[];
  vehicles: Vehicle[];
  projectiles: Projectile[];
  
  // Selection
  selectedAgentIds: string[];
  hoveredEntityId: string | null;
  
  // Time
  isPaused: boolean;
  gameSpeed: number;
  missionTime: number;
}
```

### 3D Rendering Approach

**Low-Poly Wireframe with SVGA Shading:**

```typescript
// engine/renderer/MaterialFactory.ts
import * as THREE from 'three';

export const createSVGAMaterial = (
  baseColor: number,
  wireframe: boolean = false
): THREE.MeshLambertMaterial => {
  return new THREE.MeshLambertMaterial({
    color: baseColor,
    flatShading: true,      // SVGA-style flat shading
    wireframe: wireframe,
    vertexColors: true,     // Per-vertex coloring
    emissive: 0x111111,     // Slight glow for cyberpunk feel
  });
};

// Wireframe overlay for that classic look
export const createWireframeOverlay = (
  geometry: THREE.BufferGeometry
): THREE.LineSegments => {
  const edges = new THREE.EdgesGeometry(geometry, 15);
  const lineMaterial = new THREE.LineBasicMaterial({ 
    color: 0x00ffff,
    transparent: true,
    opacity: 0.3 
  });
  return new THREE.LineSegments(edges, lineMaterial);
};
```

**Model Specifications:**

| Entity Type | Max Triangles | Max Vertices |
|-------------|---------------|--------------|
| Agent | 200 | 120 |
| Civilian | 100 | 60 |
| Vehicle | 300 | 180 |
| Building (small) | 50 | 30 |
| Building (large) | 200 | 120 |
| Weapon | 30 | 20 |

**Color Palette (Cyberpunk SVGA):**

```typescript
// data/colors.ts
export const COLORS = {
  // Factions
  EUROCORP: 0x0066cc,
  CHURCH: 0xcc0066,
  CIVILIAN: 0x666666,
  
  // Environment
  GROUND: 0x1a1a2e,
  BUILDING: 0x2d2d44,
  ROAD: 0x0f0f1a,
  
  // Effects
  LASER: 0xff0000,
  PLASMA: 0x00ffff,
  EXPLOSION: 0xff6600,
  FIRE: 0xff3300,
  
  // UI
  SELECTION: 0x00ff00,
  HOSTILE: 0xff0000,
  FRIENDLY: 0x00ff00,
  NEUTRAL: 0xffff00,
};
```

### Camera System

**Isometric Perspective:**

```typescript
// engine/renderer/CameraController.ts
export const setupIsometricCamera = (
  scene: THREE.Scene
): THREE.OrthographicCamera => {
  const frustumSize = 50;
  const aspect = window.innerWidth / window.innerHeight;
  
  const camera = new THREE.OrthographicCamera(
    -frustumSize * aspect / 2,
    frustumSize * aspect / 2,
    frustumSize / 2,
    -frustumSize / 2,
    0.1,
    1000
  );
  
  // Classic isometric angle (30 degrees)
  camera.position.set(50, 50, 50);
  camera.lookAt(0, 0, 0);
  camera.zoom = 1;
  
  return camera;
};
```

**Camera Controls:**
- WASD or Arrow keys to pan
- Mouse wheel to zoom (0.5x - 3x)
- Middle mouse drag to pan
- Home key to center on selected agents

### Pathfinding System

**A* Implementation with Navigation Mesh:**

```typescript
// engine/pathfinding/NavigationMesh.ts
interface NavNode {
  id: string;
  position: Vector3;
  neighbors: string[];
  walkable: boolean;
  cost: number; // Terrain modifier
}

interface NavMesh {
  nodes: Map<string, NavNode>;
  gridSize: number;
  
  findPath(start: Vector3, end: Vector3): Vector3[];
  updateNode(position: Vector3, walkable: boolean): void;
  rebuildArea(bounds: BoundingBox): void;
}
```

**Path Smoothing:**
- Funnel algorithm for natural movement
- Line-of-sight shortcuts
- Dynamic obstacle avoidance

### Physics System

**Collision Layers:**

```typescript
// engine/physics/CollisionLayers.ts
export enum CollisionLayer {
  NONE = 0,
  GROUND = 1 << 0,
  AGENT = 1 << 1,
  ENEMY = 1 << 2,
  CIVILIAN = 1 << 3,
  VEHICLE = 1 << 4,
  STRUCTURE = 1 << 5,
  PROJECTILE = 1 << 6,
  PICKUP = 1 << 7,
}

export const COLLISION_MATRIX = {
  [CollisionLayer.AGENT]: 
    CollisionLayer.GROUND | 
    CollisionLayer.ENEMY | 
    CollisionLayer.STRUCTURE | 
    CollisionLayer.VEHICLE,
  [CollisionLayer.PROJECTILE]:
    CollisionLayer.AGENT |
    CollisionLayer.ENEMY |
    CollisionLayer.CIVILIAN |
    CollisionLayer.VEHICLE |
    CollisionLayer.STRUCTURE,
  // ... etc
};
```

### Entity Component System

**Base Entity:**

```typescript
// types/Entity.ts
interface Entity {
  id: string;
  type: EntityType;
  position: Vector3;
  rotation: Quaternion;
  velocity: Vector3;
  
  // Component flags
  hasPhysics: boolean;
  hasAI: boolean;
  hasHealth: boolean;
  isSelectable: boolean;
  isTargetable: boolean;
}

interface HealthComponent {
  current: number;
  max: number;
  armor: number;
  regeneration: number;
}

interface CombatComponent {
  weapons: Weapon[];
  activeWeaponIndex: number;
  target: string | null;
  lastFireTime: number;
}
```

---

## Data Models

### Agent

```typescript
// types/Agent.ts
interface Agent {
  id: string;
  name: string;
  
  // Stats
  health: HealthComponent;
  
  // IPA System
  ipa: {
    intelligence: number;  // 0-100
    perception: number;    // 0-100
    adrenaline: number;    // 0-100
  };
  
  // Cyberware
  cyberware: {
    legs: CyberwareLevel;    // 0-3
    arms: CyberwareLevel;    // 0-3
    torso: CyberwareLevel;   // 0-3
    brain: CyberwareLevel;   // 0-3
    eyes: CyberwareLevel;    // 0-3
  };
  
  // Equipment
  weapons: Weapon[];
  activeWeaponIndex: number;
  equipment: Equipment[];
  
  // State
  position: Vector3;
  rotation: number;
  stance: 'idle' | 'moving' | 'combat' | 'dead';
  currentOrder: Order | null;
  
  // Persuadertron
  persuasionPower: number;
  converts: string[]; // IDs of converted units
}

type CyberwareLevel = 0 | 1 | 2 | 3;
```

### Weapon

```typescript
// types/Weapon.ts
interface Weapon {
  id: string;
  type: WeaponType;
  name: string;
  
  // Combat stats
  damage: number;
  damageType: 'kinetic' | 'energy' | 'explosive' | 'fire' | 'psychic';
  rateOfFire: number;      // rounds per second
  accuracy: number;        // base accuracy 0-1
  range: number;           // max effective range
  
  // Ammo
  ammoType: string;
  magSize: number;
  currentAmmo: number;
  reloadTime: number;
  
  // Special properties
  areaOfEffect: number;    // 0 for single target
  penetration: number;     // armor bypass
  stunChance: number;
  igniteChance: number;
  
  // Visual
  projectileSpeed: number; // 0 for hitscan
  muzzleFlash: boolean;
  tracerColor: number;
}

type WeaponType = 
  | 'pistol' | 'uzi' | 'shotgun'
  | 'minigun' | 'flamer' | 'electron_mace'
  | 'launcher' | 'high_explosive' | 'nuclear_grenade'
  | 'laser' | 'pulse_laser' | 'graviton_gun'
  | 'psycho_gas' | 'satellite_rain';
```

### Structure

```typescript
// types/Structure.ts
interface Structure {
  id: string;
  type: StructureType;
  
  // Physical
  position: Vector3;
  rotation: number;
  bounds: BoundingBox;
  
  // Destructible
  health: number;
  maxHealth: number;
  durability: 'light' | 'medium' | 'heavy' | 'indestructible';
  
  // State
  isDestroyed: boolean;
  damageLevel: number; // 0-1 for visual damage
  
  // Navigation
  blocksMovement: boolean;
  blocksProjectiles: boolean;
  
  // Interaction
  isEnterable: boolean;
  coverValue: number; // Damage reduction when behind
}

type StructureType = 
  | 'wall' | 'door' | 'fence'
  | 'building_small' | 'building_medium' | 'building_large'
  | 'bridge' | 'tower' | 'terminal';
```

### Mission

```typescript
// types/Mission.ts
interface Mission {
  id: string;
  name: string;
  briefing: string;
  
  // Location
  territory: string;
  mapId: string;
  
  // Objectives
  primaryObjectives: Objective[];
  secondaryObjectives: Objective[];
  
  // Rewards
  baseFunds: number;
  researchItems: string[];
  
  // Difficulty
  difficulty: 1 | 2 | 3 | 4 | 5;
  timeLimit: number | null;
  
  // Forces
  enemySquads: EnemySquad[];
  civilianCount: number;
  vehicleSpawns: VehicleSpawn[];
}

interface Objective {
  id: string;
  type: 'eliminate' | 'extract' | 'destroy' | 'protect' | 'capture';
  targetId: string;
  description: string;
  isComplete: boolean;
  isRequired: boolean;
}
```

### Research

```typescript
// types/Research.ts
interface ResearchProject {
  id: string;
  name: string;
  description: string;
  
  // Requirements
  prerequisites: string[];
  capturedItemRequired: string | null;
  
  // Progress
  complexity: number;       // Base research days
  progress: number;         // Current progress 0-1
  scientistsAssigned: number;
  
  // Unlock
  category: 'weapon' | 'cyberware' | 'vehicle' | 'special';
  unlocksItem: string;
  
  // Cost
  fundsCost: number;
}

interface ResearchState {
  completedProjects: string[];
  activeProjects: ResearchProject[];
  availableProjects: ResearchProject[];
  capturedItems: string[];
  totalScientists: number;
}
```

---

## Component Specifications

### Main Game Component

```typescript
// components/Game.tsx
import React, { useEffect, useRef } from 'react';
import { useGameStore } from '../state/store';
import { GameRenderer } from '../engine/renderer/GameRenderer';
import { GameLoop } from '../engine/GameLoop';
import { HUD } from './hud/HUD';
import { PauseMenu } from './menus/PauseMenu';

export const Game: React.FC = () => {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const phase = useGameStore(state => state.phase);
  const isPaused = useGameStore(state => state.isPaused);
  
  useEffect(() => {
    if (!canvasRef.current) return;
    
    const renderer = new GameRenderer(canvasRef.current);
    const gameLoop = new GameLoop(renderer);
    
    gameLoop.start();
    
    return () => {
      gameLoop.stop();
      renderer.dispose();
    };
  }, []);
  
  return (
    <div className="game-container">
      <canvas ref={canvasRef} />
      {phase === 'tactical' && <HUD />}
      {isPaused && <PauseMenu />}
    </div>
  );
};
```

### HUD Components

```typescript
// components/hud/HUD.tsx
export const HUD: React.FC = () => {
  return (
    <div className="hud">
      <AgentPanel />
      <Minimap />
      <WeaponBar />
      <ObjectiveTracker />
      <ResourceDisplay />
    </div>
  );
};

// components/hud/AgentPanel.tsx
// Displays selected agent(s) info, IPA sliders, health bars
// Max 150 lines - extract IPASliders to separate component

// components/hud/IPASliders.tsx
// Three sliders for Intelligence, Perception, Adrenaline
// Includes preset buttons

// components/hud/Minimap.tsx  
// Top-down view of mission area
// Shows agents, enemies, objectives

// components/hud/WeaponBar.tsx
// Currently equipped weapons for selected agent
// Ammo counts, switch indicators

// components/hud/ObjectiveTracker.tsx
// List of mission objectives with completion status
```

### Agent Panel Detail

```typescript
// components/hud/AgentPanel.tsx
interface AgentPanelProps {}

export const AgentPanel: React.FC<AgentPanelProps> = () => {
  const selectedAgentIds = useGameStore(state => state.selectedAgentIds);
  const agents = useGameStore(state => 
    state.agents.filter(a => selectedAgentIds.includes(a.id))
  );
  
  if (agents.length === 0) return null;
  
  if (agents.length === 1) {
    return <SingleAgentPanel agent={agents[0]} />;
  }
  
  return <MultiAgentPanel agents={agents} />;
};

// components/hud/SingleAgentPanel.tsx
export const SingleAgentPanel: React.FC<{agent: Agent}> = ({ agent }) => {
  return (
    <div className="agent-panel single">
      <AgentPortrait agent={agent} />
      <HealthBar health={agent.health} />
      <IPASliders agentId={agent.id} ipa={agent.ipa} />
      <CyberwareDisplay cyberware={agent.cyberware} />
      <WeaponSlots weapons={agent.weapons} active={agent.activeWeaponIndex} />
    </div>
  );
};
```

### IPA Sliders Component

```typescript
// components/hud/IPASliders.tsx
interface IPASlidersProps {
  agentId: string;
  ipa: { intelligence: number; perception: number; adrenaline: number };
}

export const IPASliders: React.FC<IPASlidersProps> = ({ agentId, ipa }) => {
  const setIPA = useGameStore(state => state.setAgentIPA);
  
  const handleChange = (drug: keyof typeof ipa, value: number) => {
    setIPA(agentId, drug, value);
  };
  
  const applyPreset = (preset: 'combat' | 'stealth' | 'assault') => {
    const presets = {
      combat: { intelligence: 30, perception: 70, adrenaline: 60 },
      stealth: { intelligence: 70, perception: 90, adrenaline: 20 },
      assault: { intelligence: 20, perception: 40, adrenaline: 100 },
    };
    Object.entries(presets[preset]).forEach(([drug, value]) => {
      setIPA(agentId, drug as keyof typeof ipa, value);
    });
  };
  
  return (
    <div className="ipa-sliders">
      <IPASlider 
        label="INT" 
        value={ipa.intelligence} 
        color="#0088ff"
        onChange={(v) => handleChange('intelligence', v)} 
      />
      <IPASlider 
        label="PER" 
        value={ipa.perception} 
        color="#ffcc00"
        onChange={(v) => handleChange('perception', v)} 
      />
      <IPASlider 
        label="ADR" 
        value={ipa.adrenaline} 
        color="#ff3300"
        onChange={(v) => handleChange('adrenaline', v)} 
      />
      <div className="ipa-presets">
        <button onClick={() => applyPreset('combat')}>Combat</button>
        <button onClick={() => applyPreset('stealth')}>Stealth</button>
        <button onClick={() => applyPreset('assault')}>Assault</button>
      </div>
    </div>
  );
};
```

---

## Engine Implementation

### Game Loop

```typescript
// engine/GameLoop.ts
export class GameLoop {
  private renderer: GameRenderer;
  private lastTime: number = 0;
  private accumulator: number = 0;
  private readonly FIXED_TIMESTEP = 1000 / 60; // 60 Hz physics
  private running: boolean = false;
  
  constructor(renderer: GameRenderer) {
    this.renderer = renderer;
  }
  
  start(): void {
    this.running = true;
    this.lastTime = performance.now();
    requestAnimationFrame(this.loop);
  }
  
  stop(): void {
    this.running = false;
  }
  
  private loop = (currentTime: number): void => {
    if (!this.running) return;
    
    const deltaTime = currentTime - this.lastTime;
    this.lastTime = currentTime;
    
    const store = useGameStore.getState();
    if (!store.isPaused) {
      this.accumulator += deltaTime * store.gameSpeed;
      
      // Fixed timestep updates
      while (this.accumulator >= this.FIXED_TIMESTEP) {
        this.fixedUpdate(this.FIXED_TIMESTEP / 1000);
        this.accumulator -= this.FIXED_TIMESTEP;
      }
      
      // Variable timestep render
      const alpha = this.accumulator / this.FIXED_TIMESTEP;
      this.renderer.render(alpha);
    }
    
    requestAnimationFrame(this.loop);
  };
  
  private fixedUpdate(dt: number): void {
    // Update systems in order
    InputSystem.update();
    AISystem.update(dt);
    CombatSystem.update(dt);
    PhysicsSystem.update(dt);
    SquadSystem.update(dt);
    ProjectileSystem.update(dt);
    EffectsSystem.update(dt);
  }
}
```

### Renderer

```typescript
// engine/renderer/GameRenderer.ts
import * as THREE from 'three';
import { useGameStore } from '../../state/store';
import { setupIsometricCamera } from './CameraController';
import { createSVGAMaterial, createWireframeOverlay } from './MaterialFactory';

export class GameRenderer {
  private scene: THREE.Scene;
  private camera: THREE.OrthographicCamera;
  private renderer: THREE.WebGLRenderer;
  private entityMeshes: Map<string, THREE.Object3D> = new Map();
  
  constructor(canvas: HTMLCanvasElement) {
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(0x0a0a14);
    
    this.camera = setupIsometricCamera(this.scene);
    
    this.renderer = new THREE.WebGLRenderer({ 
      canvas, 
      antialias: true 
    });
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    
    this.setupLighting();
    this.setupResizeHandler();
  }
  
  private setupLighting(): void {
    // Ambient for base visibility
    const ambient = new THREE.AmbientLight(0x404040, 0.5);
    this.scene.add(ambient);
    
    // Directional for SVGA-style shading
    const directional = new THREE.DirectionalLight(0xffffff, 0.8);
    directional.position.set(50, 100, 50);
    this.scene.add(directional);
    
    // Colored point lights for cyberpunk feel
    const blueLight = new THREE.PointLight(0x0066ff, 0.5, 100);
    blueLight.position.set(-30, 20, 0);
    this.scene.add(blueLight);
    
    const pinkLight = new THREE.PointLight(0xff0066, 0.5, 100);
    pinkLight.position.set(30, 20, 0);
    this.scene.add(pinkLight);
  }
  
  render(alpha: number): void {
    const state = useGameStore.getState();
    
    // Sync entity meshes with game state
    this.syncEntities(state.agents, 'agent');
    this.syncEntities(state.enemies, 'enemy');
    this.syncEntities(state.civilians, 'civilian');
    this.syncEntities(state.vehicles, 'vehicle');
    this.syncEntities(state.structures, 'structure');
    this.syncEntities(state.projectiles, 'projectile');
    
    // Interpolate positions for smooth rendering
    this.interpolatePositions(alpha);
    
    // Update selection indicators
    this.updateSelectionVisuals(state.selectedAgentIds);
    
    this.renderer.render(this.scene, this.camera);
  }
  
  dispose(): void {
    this.renderer.dispose();
    // Cleanup meshes, geometries, materials
  }
}
```

### Model Factory

```typescript
// models/ModelFactory.ts
import * as THREE from 'three';
import { createSVGAMaterial, createWireframeOverlay } from '../engine/renderer/MaterialFactory';
import { COLORS } from '../data/colors';

export class ModelFactory {
  private static geometryCache: Map<string, THREE.BufferGeometry> = new Map();
  
  static createAgent(faction: 'eurocorp' | 'church'): THREE.Group {
    const group = new THREE.Group();
    
    // Body (low-poly humanoid)
    const bodyGeo = this.getCachedGeometry('agent_body', () => 
      this.createLowPolyHumanoid()
    );
    const bodyMat = createSVGAMaterial(
      faction === 'eurocorp' ? COLORS.EUROCORP : COLORS.CHURCH
    );
    const body = new THREE.Mesh(bodyGeo, bodyMat);
    
    // Wireframe overlay
    const wireframe = createWireframeOverlay(bodyGeo);
    
    group.add(body);
    group.add(wireframe);
    
    return group;
  }
  
  static createBuilding(
    type: 'small' | 'medium' | 'large'
  ): THREE.Group {
    const group = new THREE.Group();
    
    const sizes = { small: 4, medium: 8, large: 16 };
    const size = sizes[type];
    
    // Simple box with beveled edges
    const geo = new THREE.BoxGeometry(size, size * 1.5, size);
    const mat = createSVGAMaterial(COLORS.BUILDING);
    const mesh = new THREE.Mesh(geo, mat);
    
    // Wireframe
    const wireframe = createWireframeOverlay(geo);
    
    // Windows (vertex coloring)
    this.addWindowLights(geo);
    
    group.add(mesh);
    group.add(wireframe);
    
    return group;
  }
  
  private static createLowPolyHumanoid(): THREE.BufferGeometry {
    // Construct low-poly humanoid from primitives
    // Head: octahedron
    // Torso: box
    // Limbs: thin boxes
    // Total ~150 triangles
    
    const geometry = new THREE.BufferGeometry();
    // ... vertex construction
    return geometry;
  }
  
  private static getCachedGeometry(
    key: string, 
    factory: () => THREE.BufferGeometry
  ): THREE.BufferGeometry {
    if (!this.geometryCache.has(key)) {
      this.geometryCache.set(key, factory());
    }
    return this.geometryCache.get(key)!;
  }
}
```

---

## Systems Implementation

### Combat System

```typescript
// systems/combat/CombatSystem.ts
import { useGameStore } from '../../state/store';
import { calculateDamage } from './DamageCalculator';
import { createProjectile } from './ProjectileFactory';

export const CombatSystem = {
  update(dt: number): void {
    const state = useGameStore.getState();
    
    // Process agent attacks
    for (const agent of state.agents) {
      if (agent.stance !== 'combat' || !agent.currentTarget) continue;
      this.processAttack(agent, dt);
    }
    
    // Process enemy attacks
    for (const enemy of state.enemies) {
      if (!enemy.target) continue;
      this.processAttack(enemy, dt);
    }
  },
  
  processAttack(attacker: CombatEntity, dt: number): void {
    const weapon = attacker.weapons[attacker.activeWeaponIndex];
    if (!weapon) return;
    
    const timeSinceLastFire = Date.now() - attacker.lastFireTime;
    const fireInterval = 1000 / weapon.rateOfFire;
    
    if (timeSinceLastFire < fireInterval) return;
    
    // Calculate accuracy with IPA modifiers
    const accuracy = this.calculateAccuracy(attacker, weapon);
    
    if (weapon.projectileSpeed === 0) {
      // Hitscan weapon
      this.processHitscan(attacker, weapon, accuracy);
    } else {
      // Projectile weapon
      const projectile = createProjectile(attacker, weapon);
      useGameStore.getState().addProjectile(projectile);
    }
    
    // Update fire time and ammo
    useGameStore.getState().updateAttacker(attacker.id, {
      lastFireTime: Date.now(),
      currentAmmo: weapon.currentAmmo - 1
    });
  },
  
  calculateAccuracy(attacker: CombatEntity, weapon: Weapon): number {
    let accuracy = weapon.accuracy;
    
    // IPA modifiers (if agent)
    if ('ipa' in attacker) {
      accuracy *= (0.5 + attacker.ipa.intelligence / 200);
      accuracy *= (1 - attacker.ipa.adrenaline / 400);
    }
    
    // Cyberware bonuses
    if ('cyberware' in attacker) {
      accuracy *= (1 + attacker.cyberware.arms * 0.1);
    }
    
    return Math.min(1, accuracy);
  }
};
```

### Damage Calculator

```typescript
// systems/combat/DamageCalculator.ts
interface DamageResult {
  finalDamage: number;
  isCritical: boolean;
  damageType: string;
  statusEffects: StatusEffect[];
}

export const calculateDamage = (
  weapon: Weapon,
  target: HealthComponent,
  distance: number
): DamageResult => {
  let damage = weapon.damage;
  const statusEffects: StatusEffect[] = [];
  
  // Distance falloff
  if (distance > weapon.range * 0.5) {
    const falloff = 1 - ((distance - weapon.range * 0.5) / (weapon.range * 0.5));
    damage *= Math.max(0.25, falloff);
  }
  
  // Armor reduction
  const armorReduction = target.armor * (1 - weapon.penetration);
  damage = Math.max(1, damage - armorReduction);
  
  // Critical hit (10% base chance)
  const isCritical = Math.random() < 0.1;
  if (isCritical) damage *= 2;
  
  // Status effects
  if (weapon.stunChance > 0 && Math.random() < weapon.stunChance) {
    statusEffects.push({ type: 'stun', duration: 2 });
  }
  if (weapon.igniteChance > 0 && Math.random() < weapon.igniteChance) {
    statusEffects.push({ type: 'burning', duration: 5, damagePerSecond: 5 });
  }
  
  return {
    finalDamage: Math.round(damage),
    isCritical,
    damageType: weapon.damageType,
    statusEffects
  };
};
```

### AI System

```typescript
// systems/ai/AISystem.ts
import { useGameStore } from '../../state/store';
import { BehaviorTree, BehaviorStatus } from './BehaviorTree';
import { createEnemyBehavior } from './EnemyBehaviors';

export const AISystem = {
  behaviorTrees: new Map<string, BehaviorTree>(),
  
  update(dt: number): void {
    const state = useGameStore.getState();
    
    for (const enemy of state.enemies) {
      if (enemy.health.current <= 0) continue;
      
      // Get or create behavior tree
      let bt = this.behaviorTrees.get(enemy.id);
      if (!bt) {
        bt = createEnemyBehavior(enemy.type);
        this.behaviorTrees.set(enemy.id, bt);
      }
      
      // Execute behavior
      bt.tick(enemy, state, dt);
    }
    
    // Update converted civilians
    for (const civilian of state.civilians.filter(c => c.isConverted)) {
      this.updateConvertedBehavior(civilian, dt);
    }
  },
  
  updateConvertedBehavior(civilian: Civilian, dt: number): void {
    // Follow master agent
    const state = useGameStore.getState();
    const master = state.agents.find(a => a.converts.includes(civilian.id));
    if (!master) return;
    
    const distance = civilian.position.distanceTo(master.position);
    if (distance > 5) {
      // Move toward master
      const direction = master.position.clone()
        .sub(civilian.position)
        .normalize();
      civilian.position.add(direction.multiplyScalar(3 * dt));
    }
  }
};
```

### Enemy Behaviors

```typescript
// systems/ai/EnemyBehaviors.ts
import { 
  BehaviorTree, 
  Selector, 
  Sequence, 
  Condition, 
  Action 
} from './BehaviorTree';

export const createEnemyBehavior = (type: EnemyType): BehaviorTree => {
  return new BehaviorTree(
    new Selector([
      // Priority 1: Flee if critical health
      new Sequence([
        new Condition('health_critical', (e) => e.health.current < e.health.max * 0.2),
        new Action('flee', fleeFromThreats)
      ]),
      
      // Priority 2: Attack visible enemies
      new Sequence([
        new Condition('has_target', (e, state) => !!findVisibleEnemy(e, state)),
        new Action('engage', engageTarget)
      ]),
      
      // Priority 3: Investigate sounds
      new Sequence([
        new Condition('heard_noise', (e, state) => !!e.lastHeardNoise),
        new Action('investigate', investigateNoise)
      ]),
      
      // Priority 4: Patrol
      new Action('patrol', patrolRoute)
    ])
  );
};

const engageTarget = (enemy: Enemy, state: GameState, dt: number): BehaviorStatus => {
  const target = findVisibleEnemy(enemy, state);
  if (!target) return BehaviorStatus.FAILURE;
  
  const distance = enemy.position.distanceTo(target.position);
  const weapon = enemy.weapons[enemy.activeWeaponIndex];
  
  if (distance > weapon.range) {
    // Move closer
    moveToward(enemy, target.position, dt);
    return BehaviorStatus.RUNNING;
  }
  
  // In range - attack
  enemy.currentTarget = target.id;
  enemy.stance = 'combat';
  return BehaviorStatus.SUCCESS;
};
```

### Squad System

```typescript
// systems/squad/SquadSystem.ts
import { useGameStore } from '../../state/store';
import { PathfindingSystem } from '../../engine/pathfinding/PathfindingSystem';

export const SquadSystem = {
  update(dt: number): void {
    const state = useGameStore.getState();
    
    for (const agent of state.agents) {
      if (agent.health.current <= 0) continue;
      
      // Apply IPA effects
      this.applyIPAEffects(agent, dt);
      
      // Process current order
      if (agent.currentOrder) {
        this.processOrder(agent, dt);
      }
      
      // Auto-acquire targets based on perception
      if (agent.stance !== 'combat') {
        this.checkForThreats(agent, state);
      }
    }
  },
  
  applyIPAEffects(agent: Agent, dt: number): void {
    const { intelligence, perception, adrenaline } = agent.ipa;
    
    // Calculate effective stats
    agent.effectiveStats = {
      accuracy: 0.5 + (intelligence / 200),
      reactionTime: 1 - (intelligence / 200),
      visionRange: 10 + (perception / 5),
      moveSpeed: 3 + (adrenaline / 25),
      stability: 1 - (adrenaline / 200)
    };
  },
  
  processOrder(agent: Agent, dt: number): void {
    const order = agent.currentOrder;
    
    switch (order.type) {
      case 'move':
        this.processMove(agent, order, dt);
        break;
      case 'attack':
        this.processAttack(agent, order, dt);
        break;
      case 'use':
        this.processUse(agent, order, dt);
        break;
      case 'enter_vehicle':
        this.processEnterVehicle(agent, order);
        break;
    }
  },
  
  processMove(agent: Agent, order: MoveOrder, dt: number): void {
    if (!order.path || order.path.length === 0) {
      // Calculate path
      order.path = PathfindingSystem.findPath(
        agent.position, 
        order.destination
      );
    }
    
    const nextWaypoint = order.path[0];
    const distance = agent.position.distanceTo(nextWaypoint);
    
    if (distance < 0.5) {
      order.path.shift();
      if (order.path.length === 0) {
        agent.currentOrder = null;
        agent.stance = 'idle';
      }
      return;
    }
    
    // Move toward waypoint
    const direction = nextWaypoint.clone()
      .sub(agent.position)
      .normalize();
    const moveSpeed = agent.effectiveStats.moveSpeed;
    agent.position.add(direction.multiplyScalar(moveSpeed * dt));
    agent.stance = 'moving';
    
    // Face movement direction
    agent.rotation = Math.atan2(direction.x, direction.z);
  }
};
```

---

## Input Handling

### Input System

```typescript
// systems/input/InputSystem.ts
import { useGameStore } from '../../state/store';
import { Raycaster, Vector2, Vector3 } from 'three';

export const InputSystem = {
  raycaster: new Raycaster(),
  mouse: new Vector2(),
  
  init(canvas: HTMLCanvasElement): void {
    canvas.addEventListener('click', this.handleClick);
    canvas.addEventListener('contextmenu', this.handleRightClick);
    canvas.addEventListener('mousemove', this.handleMouseMove);
    window.addEventListener('keydown', this.handleKeyDown);
    window.addEventListener('keyup', this.handleKeyUp);
  },
  
  handleClick = (event: MouseEvent): void => {
    const hit = this.raycast(event);
    if (!hit) return;
    
    const state = useGameStore.getState();
    
    if (hit.entity) {
      if (hit.entity.type === 'agent') {
        // Select agent
        if (event.ctrlKey) {
          state.toggleAgentSelection(hit.entity.id);
        } else {
          state.setSelectedAgents([hit.entity.id]);
        }
      }
    } else {
      // Deselect all
      state.setSelectedAgents([]);
    }
  },
  
  handleRightClick = (event: MouseEvent): void => {
    event.preventDefault();
    const hit = this.raycast(event);
    if (!hit) return;
    
    const state = useGameStore.getState();
    const selectedAgents = state.agents.filter(
      a => state.selectedAgentIds.includes(a.id)
    );
    
    if (selectedAgents.length === 0) return;
    
    if (hit.entity && hit.entity.faction === 'enemy') {
      // Attack order
      for (const agent of selectedAgents) {
        state.issueOrder(agent.id, {
          type: 'attack',
          targetId: hit.entity.id
        });
      }
    } else if (hit.point) {
      // Move order
      const formation = this.calculateFormation(
        selectedAgents.length, 
        hit.point
      );
      
      selectedAgents.forEach((agent, i) => {
        state.issueOrder(agent.id, {
          type: 'move',
          destination: formation[i],
          path: null
        });
      });
    }
  },
  
  handleKeyDown = (event: KeyboardEvent): void => {
    const state = useGameStore.getState();
    
    switch (event.key) {
      case '1': case '2': case '3': case '4':
        const index = parseInt(event.key) - 1;
        const agent = state.agents[index];
        if (agent) {
          if (event.ctrlKey) {
            state.toggleAgentSelection(agent.id);
          } else {
            state.setSelectedAgents([agent.id]);
          }
        }
        break;
        
      case '5':
        state.setSelectedAgents(state.agents.map(a => a.id));
        break;
        
      case 'h':
      case 'H':
        // Hold position
        for (const id of state.selectedAgentIds) {
          state.issueOrder(id, { type: 'hold' });
        }
        break;
        
      case 'Escape':
        state.togglePause();
        break;
    }
  },
  
  calculateFormation(count: number, center: Vector3): Vector3[] {
    // Box formation
    const spacing = 2;
    const positions: Vector3[] = [];
    const cols = Math.ceil(Math.sqrt(count));
    
    for (let i = 0; i < count; i++) {
      const row = Math.floor(i / cols);
      const col = i % cols;
      positions.push(new Vector3(
        center.x + (col - cols / 2) * spacing,
        center.y,
        center.z + row * spacing
      ));
    }
    
    return positions;
  }
};
```

---

## State Store

### Zustand Store

```typescript
// state/store.ts
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';
import { GameState, Agent, Order } from '../types';

interface GameStore extends GameState {
  // Selection actions
  setSelectedAgents: (ids: string[]) => void;
  toggleAgentSelection: (id: string) => void;
  
  // Order actions
  issueOrder: (agentId: string, order: Order) => void;
  
  // IPA actions
  setAgentIPA: (agentId: string, drug: string, value: number) => void;
  
  // Combat actions
  addProjectile: (projectile: Projectile) => void;
  removeProjectile: (id: string) => void;
  damageEntity: (id: string, damage: number) => void;
  
  // Game flow
  togglePause: () => void;
  setGameSpeed: (speed: number) => void;
  
  // Mission
  loadMission: (missionId: string) => void;
  completeObjective: (objectiveId: string) => void;
}

export const useGameStore = create<GameStore>()(
  immer((set, get) => ({
    // Initial state
    phase: 'menu',
    campaign: 'eurocorp',
    territories: [],
    research: initialResearchState,
    funds: 50000,
    agents: [],
    enemies: [],
    civilians: [],
    structures: [],
    vehicles: [],
    projectiles: [],
    selectedAgentIds: [],
    hoveredEntityId: null,
    isPaused: false,
    gameSpeed: 1,
    missionTime: 0,
    
    // Actions
    setSelectedAgents: (ids) => set((state) => {
      state.selectedAgentIds = ids;
    }),
    
    toggleAgentSelection: (id) => set((state) => {
      const index = state.selectedAgentIds.indexOf(id);
      if (index >= 0) {
        state.selectedAgentIds.splice(index, 1);
      } else {
        state.selectedAgentIds.push(id);
      }
    }),
    
    issueOrder: (agentId, order) => set((state) => {
      const agent = state.agents.find(a => a.id === agentId);
      if (agent) {
        agent.currentOrder = order;
      }
    }),
    
    setAgentIPA: (agentId, drug, value) => set((state) => {
      const agent = state.agents.find(a => a.id === agentId);
      if (agent && drug in agent.ipa) {
        agent.ipa[drug as keyof typeof agent.ipa] = 
          Math.max(0, Math.min(100, value));
      }
    }),
    
    damageEntity: (id, damage) => set((state) => {
      // Find entity in any collection
      const collections = ['agents', 'enemies', 'civilians', 'vehicles'];
      for (const collection of collections) {
        const entity = state[collection].find((e: any) => e.id === id);
        if (entity?.health) {
          entity.health.current = Math.max(0, entity.health.current - damage);
          if (entity.health.current <= 0) {
            entity.stance = 'dead';
          }
          break;
        }
      }
    }),
    
    togglePause: () => set((state) => {
      state.isPaused = !state.isPaused;
    }),
    
    // ... additional actions
  }))
);
```

---

## Styling

### CSS Architecture

```css
/* styles/main.css */
:root {
  /* Cyberpunk color palette */
  --color-bg: #0a0a14;
  --color-panel: #12121f;
  --color-border: #2a2a44;
  --color-text: #e0e0ff;
  --color-accent-blue: #0088ff;
  --color-accent-pink: #ff0088;
  --color-accent-cyan: #00ffff;
  --color-eurocorp: #0066cc;
  --color-church: #cc0066;
  --color-health: #00ff66;
  --color-danger: #ff3300;
  
  /* Spacing */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  
  /* Typography */
  --font-main: 'Orbitron', 'Consolas', monospace;
  --font-size-sm: 12px;
  --font-size-md: 14px;
  --font-size-lg: 18px;
}

.game-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: var(--color-bg);
}

canvas {
  display: block;
  width: 100%;
  height: 100%;
}

/* HUD Layer */
.hud {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  font-family: var(--font-main);
  color: var(--color-text);
}

.hud > * {
  pointer-events: auto;
}
```

### Agent Panel Styles

```css
/* styles/components/agent-panel.css */
.agent-panel {
  position: absolute;
  bottom: var(--spacing-md);
  left: var(--spacing-md);
  background: var(--color-panel);
  border: 1px solid var(--color-border);
  border-radius: 4px;
  padding: var(--spacing-md);
  min-width: 280px;
}

.agent-panel.single {
  display: grid;
  grid-template-columns: 80px 1fr;
  grid-template-rows: auto auto auto;
  gap: var(--spacing-sm);
}

.agent-portrait {
  grid-row: span 2;
  width: 80px;
  height: 80px;
  background: var(--color-bg);
  border: 2px solid var(--color-border);
}

.health-bar {
  height: 12px;
  background: var(--color-bg);
  border-radius: 2px;
  overflow: hidden;
}

.health-bar-fill {
  height: 100%;
  background: linear-gradient(
    90deg, 
    var(--color-danger) 0%, 
    var(--color-health) 100%
  );
  transition: width 0.2s ease;
}

.ipa-sliders {
  grid-column: span 2;
  display: flex;
  gap: var(--spacing-md);
}

.ipa-slider {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.ipa-slider input[type="range"] {
  writing-mode: vertical-lr;
  direction: rtl;
  height: 100px;
  width: 24px;
}

.ipa-presets {
  display: flex;
  gap: var(--spacing-xs);
  margin-top: var(--spacing-sm);
}

.ipa-presets button {
  flex: 1;
  padding: var(--spacing-xs) var(--spacing-sm);
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  color: var(--color-text);
  font-family: var(--font-main);
  font-size: var(--font-size-sm);
  cursor: pointer;
  transition: all 0.2s;
}

.ipa-presets button:hover {
  background: var(--color-accent-blue);
  border-color: var(--color-accent-cyan);
}
```

---

## Testing Scenarios

### Unit Tests

```typescript
// __tests__/combat/DamageCalculator.test.ts
describe('DamageCalculator', () => {
  it('applies distance falloff beyond half range', () => {
    const weapon = createTestWeapon({ damage: 100, range: 20 });
    const target = createTestHealth({ armor: 0 });
    
    const closeResult = calculateDamage(weapon, target, 5);
    const farResult = calculateDamage(weapon, target, 18);
    
    expect(closeResult.finalDamage).toBe(100);
    expect(farResult.finalDamage).toBeLessThan(50);
  });
  
  it('reduces damage based on armor minus penetration', () => {
    const weapon = createTestWeapon({ damage: 100, penetration: 0.5 });
    const target = createTestHealth({ armor: 40 });
    
    const result = calculateDamage(weapon, target, 0);
    
    // 100 - (40 * 0.5) = 80
    expect(result.finalDamage).toBe(80);
  });
});

// __tests__/squad/IPASystem.test.ts
describe('IPA System', () => {
  it('increases accuracy at high intelligence', () => {
    const lowInt = calculateEffectiveAccuracy(20, 50, 50);
    const highInt = calculateEffectiveAccuracy(100, 50, 50);
    
    expect(highInt).toBeGreaterThan(lowInt);
  });
  
  it('decreases accuracy at high adrenaline', () => {
    const lowAdr = calculateEffectiveAccuracy(50, 50, 20);
    const highAdr = calculateEffectiveAccuracy(50, 50, 100);
    
    expect(lowAdr).toBeGreaterThan(highAdr);
  });
});
```

### Integration Tests

```typescript
// __tests__/integration/MissionFlow.test.ts
describe('Mission Flow', () => {
  it('completes mission when all objectives met', async () => {
    const store = createTestStore();
    store.getState().loadMission('test_assassination');
    
    const target = store.getState().enemies[0];
    store.getState().damageEntity(target.id, 1000);
    
    // Verify objective complete
    const objective = store.getState().currentMission.primaryObjectives[0];
    expect(objective.isComplete).toBe(true);
  });
  
  it('pathfinding navigates around obstacles', () => {
    const path = PathfindingSystem.findPath(
      new Vector3(0, 0, 0),
      new Vector3(10, 0, 10)
    );
    
    // Path should not intersect any obstacles
    for (const point of path) {
      const collisions = checkCollisions(point, CollisionLayer.STRUCTURE);
      expect(collisions).toHaveLength(0);
    }
  });
});
```

---

## Performance Requirements

| Metric | Target |
|--------|--------|
| Frame Rate | 60 FPS on mid-range hardware |
| Initial Load | < 3 seconds |
| Memory Usage | < 512 MB |
| Entities on Screen | 100+ simultaneously |
| Pathfinding | < 5ms for 50 tile path |
| Physics Updates | < 2ms per frame |

### Optimization Strategies

1. **Geometry Instancing** - Reuse mesh geometry for identical entities
2. **Frustum Culling** - Don't render off-screen entities
3. **LOD System** - Reduce detail for distant entities
4. **Object Pooling** - Reuse projectiles and effects
5. **Spatial Hashing** - Efficient collision detection
6. **Web Workers** - Offload pathfinding calculations

---

## Accessibility Requirements

1. **Keyboard Navigation** - All game functions accessible via keyboard
2. **Pause Anytime** - Tactical situations can be paused to issue orders
3. **Color Blind Modes** - Alternative color schemes for deuteranopia, protanopia
4. **Text Scaling** - UI text can be scaled 100%-200%
5. **Audio Cues** - Important events have distinct audio feedback
6. **Remappable Controls** - All keybindings configurable

---

## File Structure Summary

```
src/
├── components/
│   ├── Game.tsx                    (~50 lines)
│   ├── hud/
│   │   ├── HUD.tsx                 (~30 lines)
│   │   ├── AgentPanel.tsx          (~80 lines)
│   │   ├── SingleAgentPanel.tsx    (~60 lines)
│   │   ├── MultiAgentPanel.tsx     (~50 lines)
│   │   ├── IPASliders.tsx          (~90 lines)
│   │   ├── IPASlider.tsx           (~40 lines)
│   │   ├── HealthBar.tsx           (~30 lines)
│   │   ├── WeaponBar.tsx           (~60 lines)
│   │   ├── Minimap.tsx             (~100 lines)
│   │   └── ObjectiveTracker.tsx    (~50 lines)
│   ├── menus/
│   │   ├── MainMenu.tsx            (~80 lines)
│   │   ├── PauseMenu.tsx           (~60 lines)
│   │   ├── MissionSelect.tsx       (~100 lines)
│   │   └── ResearchScreen.tsx      (~150 lines)
│   └── common/
│       ├── Button.tsx              (~40 lines)
│       ├── Slider.tsx              (~50 lines)
│       └── Panel.tsx               (~30 lines)
├── engine/
│   ├── GameLoop.ts                 (~80 lines)
│   ├── renderer/
│   │   ├── GameRenderer.ts         (~150 lines)
│   │   ├── CameraController.ts     (~100 lines)
│   │   ├── MaterialFactory.ts      (~60 lines)
│   │   └── SelectionVisuals.ts     (~80 lines)
│   ├── physics/
│   │   ├── PhysicsSystem.ts        (~100 lines)
│   │   ├── CollisionLayers.ts      (~40 lines)
│   │   └── SpatialHash.ts          (~80 lines)
│   ├── pathfinding/
│   │   ├── PathfindingSystem.ts    (~120 lines)
│   │   ├── NavigationMesh.ts       (~100 lines)
│   │   └── AStar.ts                (~80 lines)
│   └── audio/
│       └── AudioSystem.ts          (~100 lines)
├── systems/
│   ├── combat/
│   │   ├── CombatSystem.ts         (~120 lines)
│   │   ├── DamageCalculator.ts     (~80 lines)
│   │   ├── ProjectileSystem.ts     (~100 lines)
│   │   └── ProjectileFactory.ts    (~60 lines)
│   ├── ai/
│   │   ├── AISystem.ts             (~100 lines)
│   │   ├── BehaviorTree.ts         (~120 lines)
│   │   └── EnemyBehaviors.ts       (~150 lines)
│   ├── squad/
│   │   ├── SquadSystem.ts          (~150 lines)
│   │   └── FormationSystem.ts      (~80 lines)
│   ├── world/
│   │   ├── DestructionSystem.ts    (~100 lines)
│   │   └── VehicleSystem.ts        (~120 lines)
│   └── input/
│       └── InputSystem.ts          (~180 lines)
├── models/
│   ├── ModelFactory.ts             (~150 lines)
│   ├── agents/
│   │   └── AgentModel.ts           (~80 lines)
│   ├── weapons/
│   │   └── WeaponModels.ts         (~100 lines)
│   ├── vehicles/
│   │   └── VehicleModels.ts        (~80 lines)
│   └── structures/
│       └── BuildingModels.ts       (~100 lines)
├── state/
│   ├── store.ts                    (~200 lines)
│   └── selectors.ts                (~60 lines)
├── data/
│   ├── colors.ts                   (~40 lines)
│   ├── weapons.ts                  (~150 lines)
│   ├── cyberware.ts                (~80 lines)
│   ├── missions.ts                 (~200 lines)
│   └── research.ts                 (~100 lines)
├── hooks/
│   ├── useGameLoop.ts              (~40 lines)
│   ├── useInput.ts                 (~60 lines)
│   └── useSelection.ts             (~50 lines)
├── types/
│   ├── index.ts                    (~20 lines)
│   ├── GameState.ts                (~60 lines)
│   ├── Entity.ts                   (~80 lines)
│   ├── Agent.ts                    (~70 lines)
│   ├── Weapon.ts                   (~50 lines)
│   ├── Structure.ts                (~40 lines)
│   ├── Mission.ts                  (~60 lines)
│   └── Research.ts                 (~50 lines)
├── utils/
│   ├── math.ts                     (~60 lines)
│   ├── random.ts                   (~30 lines)
│   └── entityHelpers.ts            (~50 lines)
├── styles/
│   ├── main.css                    (~100 lines)
│   └── components/
│       ├── agent-panel.css         (~80 lines)
│       ├── minimap.css             (~40 lines)
│       └── menus.css               (~100 lines)
├── App.tsx                         (~30 lines)
└── index.tsx                       (~20 lines)
```

**Total: ~65 files, average ~85 lines each**

---

## Implementation Order

### Phase 1: Foundation (Week 1)
1. Project setup with Vite + React + TypeScript + Three.js
2. Basic state store with Zustand
3. Game loop and renderer scaffolding
4. Camera controller (isometric view)
5. Basic input handling

### Phase 2: Core Rendering (Week 2)
1. Material factory (SVGA shading)
2. Model factory (low-poly geometries)
3. Agent model and rendering
4. Building/structure models
5. Selection visuals

### Phase 3: Movement & Navigation (Week 3)
1. Navigation mesh generation
2. A* pathfinding implementation
3. Squad movement and formations
4. Basic physics/collision

### Phase 4: Combat Systems (Week 4)
1. Weapon definitions and data
2. Combat system (shooting, damage)
3. Projectile system
4. Hitscan weapons
5. Area of effect weapons

### Phase 5: Agent Systems (Week 5)
1. IPA system implementation
2. Cyberware effects
3. Equipment and inventory
4. Persuadertron mechanics

### Phase 6: AI & Enemies (Week 6)
1. Behavior tree framework
2. Enemy behaviors (patrol, engage, flee)
3. Civilian AI
4. Converted unit following

### Phase 7: World Systems (Week 7)
1. Destructible structures
2. Vehicle system
3. Fire propagation
4. Dynamic pathfinding updates

### Phase 8: UI & Menus (Week 8)
1. HUD components (agent panel, minimap)
2. Main menu
3. Mission selection
4. Research screen
5. Pause menu

### Phase 9: Content & Polish (Week 9-10)
1. Mission definitions
2. Campaign structure
3. Research tree content
4. Sound effects and music
5. Performance optimization
6. Bug fixing and balance

---

## Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "three": "^0.160.0",
    "@react-three/fiber": "^8.15.0",
    "zustand": "^4.4.0",
    "immer": "^10.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/three": "^0.160.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "vitest": "^1.0.0"
  }
}
```

---

## Conclusion

REACT-WARS faithfully recreates the tactical depth of Syndicate Wars in a modern React/Three.js architecture. The low-poly wireframe aesthetic with SVGA shading provides distinctive visuals without texture dependencies, while the strict component-based architecture ensures maintainability and extensibility.

Key technical achievements:
- **Modular design**: No file exceeds 200 lines
- **Clean separation**: Rendering, logic, and state are fully decoupled
- **Performant**: Instanced geometry, spatial hashing, efficient state updates
- **Faithful adaptation**: All core Syndicate Wars systems recreated

The implementation follows TINS principles: this README contains sufficient detail for any capable LLM to generate a complete, functional implementation.
