# REACT-ACE

A retro-futuristic arcade flight combat game built with React Three Fiber, drei, and Three.js. Features wireframe vector graphics with SVGA-style vertex shading, wave-based survival missions, swarm missile lock-on systems, and a globe-trotting flying mega-carrier as your mobile base.

REACT-ACE distills the proven mechanics of the ACE COMBAT series into a stylized package—prioritizing responsive controls, satisfying feedback loops, and the core fantasy of being a legendary ace pilot. The wireframe aesthetic evokes classic vector games while modern Three.js rendering enables smooth 60fps gameplay with complex particle systems and dynamic lighting.

---

## Technical Stack

```
Framework:       React 18+ with TypeScript
3D Engine:       Three.js r150+
React Binding:   @react-three/fiber (R3F) 8+
Utilities:       @react-three/drei 9+
State:           Zustand 4+
Audio:           Howler.js 2.2+
Physics:         Custom arcade physics (no library)
Build:           Vite 5+
```

### Why This Stack

React Three Fiber provides declarative 3D scene composition matching React's component model. Drei supplies essential helpers (cameras, controls, effects) without reinventing fundamentals. Zustand offers minimal-boilerplate state management critical for 60fps game loops. Custom physics avoids simulation overhead—ACE COMBAT-style flight requires "feel" over accuracy.

---

## Architecture Overview

```
src/
├── index.jsx                    # App entry, providers
├── App.jsx                      # Root component, game states
│
├── core/                        # Engine systems (non-visual)
│   ├── GameLoop.js              # Fixed timestep, delta management
│   ├── InputManager.js          # Keyboard/gamepad abstraction
│   ├── AudioManager.js          # Spatial audio, music layers
│   └── PhysicsWorld.js          # Arcade flight physics
│
├── state/                       # Zustand stores
│   ├── useGameStore.js          # Master game state
│   ├── usePlayerStore.js        # Player aircraft state
│   ├── useMissionStore.js       # Wave/objective state
│   ├── useTargetStore.js        # All targetable entities
│   └── useUIStore.js            # HUD/menu state
│
├── components/                  # React components
│   ├── game/                    # 3D game components
│   │   ├── GameCanvas.jsx       # R3F Canvas wrapper
│   │   ├── Scene.jsx            # Scene composition
│   │   ├── Environment.jsx      # Sky, clouds, globe
│   │   └── PostProcessing.jsx   # Bloom, scanlines
│   │
│   ├── entities/                # Game objects
│   │   ├── PlayerAircraft.jsx   # Player fighter
│   │   ├── EnemyAircraft.jsx    # AI fighters
│   │   ├── Missile.jsx          # All missile types
│   │   ├── MegaCarrier.jsx      # Flying base
│   │   └── Explosion.jsx        # Destruction effects
│   │
│   ├── systems/                 # ECS-style systems as components
│   │   ├── FlightSystem.jsx     # Aircraft physics
│   │   ├── TargetingSystem.jsx  # Lock-on logic
│   │   ├── MissileSystem.jsx    # Missile guidance
│   │   ├── WaveSystem.jsx       # Enemy spawning
│   │   ├── ScoringSystem.jsx    # Points/ranks
│   │   └── CollisionSystem.jsx  # Hit detection
│   │
│   ├── hud/                     # 2D overlay (HTML/CSS)
│   │   ├── HUD.jsx              # HUD container
│   │   ├── Speedometer.jsx      # Speed display
│   │   ├── Altimeter.jsx        # Altitude display
│   │   ├── Radar.jsx            # Minimap radar
│   │   ├── TargetBox.jsx        # Lock-on reticles
│   │   ├── WeaponDisplay.jsx    # Ammo/weapon select
│   │   ├── MissileWarning.jsx   # Incoming alerts
│   │   └── WaveIndicator.jsx    # Current wave/timer
│   │
│   └── ui/                      # Menus (HTML/CSS)
│       ├── MainMenu.jsx         # Title screen
│       ├── Briefing.jsx         # Mission briefing
│       ├── Hangar.jsx           # Aircraft select
│       ├── PauseMenu.jsx        # Pause overlay
│       └── Results.jsx          # Mission complete
│
├── data/                        # Static game data
│   ├── aircraft.js              # Aircraft stats
│   ├── weapons.js               # Weapon definitions
│   ├── waves.js                 # Wave configurations
│   └── missions.js              # Mission definitions
│
├── hooks/                       # Custom React hooks
│   ├── useGameLoop.js           # Frame timing
│   ├── useInput.js              # Input state
│   ├── useAudio.js              # Sound triggers
│   └── useTargeting.js          # Lock-on helpers
│
├── utils/                       # Pure functions
│   ├── math.js                  # Vector/quaternion helpers
│   ├── geometry.js              # Wireframe generation
│   └── constants.js             # Game constants
│
└── shaders/                     # Custom GLSL
    ├── wireframe.vert           # Vertex shader
    ├── wireframe.frag           # Fragment shader
    └── explosion.frag           # Particle shader
```

### Modular Design Principles

**Maximum 200 lines per file.** When a file approaches this limit, extract sub-components or utility functions. This prevents cognitive overload and enables parallel development.

**Single Responsibility.** Each component handles one concern:
- `PlayerAircraft.jsx` renders the mesh and exposes refs
- `FlightSystem.jsx` updates position/rotation via store
- `TargetingSystem.jsx` manages lock-on state separately

**Data-Driven Configuration.** Aircraft stats, weapon properties, and wave definitions live in `/data/` as plain objects. Components consume these via stores, never hardcode values.

**Zustand for Cross-Component State.** Avoid prop drilling. Systems read/write stores directly. React's reconciliation handles re-renders efficiently.

---

## Core Systems

### Flight Physics

The flight model prioritizes "feel" over simulation. Aircraft have weight and momentum but ignore realistic aerodynamics.

```javascript
// PhysicsWorld.js - Core flight update (called every frame)

const GRAVITY = 9.81;
const AIR_DENSITY = 0.5; // Arbitrary drag coefficient

function updateAircraft(aircraft, input, deltaTime) {
  const { position, velocity, rotation, stats } = aircraft;
  
  // Thrust (forward acceleration)
  const thrustDir = new Vector3(0, 0, -1).applyQuaternion(rotation);
  const thrust = input.throttle * stats.maxThrust;
  velocity.addScaledVector(thrustDir, thrust * deltaTime);
  
  // Drag (speed-dependent deceleration)
  const speed = velocity.length();
  const drag = speed * speed * AIR_DENSITY * stats.dragCoeff;
  velocity.addScaledVector(velocity.clone().normalize(), -drag * deltaTime);
  
  // Lift (simplified - just counteracts gravity when moving)
  const liftFactor = Math.min(speed / stats.stallSpeed, 1.0);
  velocity.y += GRAVITY * (liftFactor - 1) * deltaTime;
  
  // Rotation (pitch, yaw, roll from input)
  const pitchRate = input.pitch * stats.pitchSpeed * deltaTime;
  const yawRate = input.yaw * stats.yawSpeed * deltaTime;
  const rollRate = input.roll * stats.rollSpeed * deltaTime;
  
  const pitchQuat = new Quaternion().setFromAxisAngle(new Vector3(1, 0, 0), pitchRate);
  const yawQuat = new Quaternion().setFromAxisAngle(new Vector3(0, 1, 0), yawRate);
  const rollQuat = new Quaternion().setFromAxisAngle(new Vector3(0, 0, 1), rollRate);
  
  rotation.multiply(rollQuat).multiply(pitchQuat).multiply(yawQuat);
  
  // Update position
  position.addScaledVector(velocity, deltaTime);
  
  // Enforce altitude limits
  position.y = Math.max(position.y, 50);  // Minimum altitude
  position.y = Math.min(position.y, 25000); // Maximum altitude
  
  return { position, velocity, rotation, speed };
}
```

#### High-G Turns

Activated when both accelerate AND decelerate inputs are held while turning. Doubles turn rate, rapidly bleeds speed.

```javascript
// In FlightSystem.jsx

const isHighGTurn = input.accelerate && input.decelerate && 
                    (Math.abs(input.pitch) > 0.3 || Math.abs(input.roll) > 0.3);

if (isHighGTurn) {
  // Double turn rates
  effectivePitchSpeed *= 2.0;
  effectiveRollSpeed *= 2.0;
  
  // Bleed speed aggressively
  const speedBleed = speed * 0.4 * deltaTime; // 40% per second
  velocity.addScaledVector(velocity.clone().normalize(), -speedBleed);
  
  // Visual feedback
  setHighGIndicator(true);
  // Audio feedback handled by AudioManager
}
```

#### Stall Behavior

Below stall speed, aircraft loses lift and control authority diminishes.

```javascript
const stallFactor = Math.max(0, (speed - stats.stallSpeed * 0.5) / (stats.stallSpeed * 0.5));

// Reduce control authority
effectivePitchSpeed *= stallFactor;
effectiveYawSpeed *= stallFactor;
effectiveRollSpeed *= stallFactor * 0.5; // Roll degrades faster

// Nose drops
if (stallFactor < 0.3) {
  const noseDrop = (1 - stallFactor) * 0.5 * deltaTime;
  rotation.multiply(new Quaternion().setFromAxisAngle(new Vector3(1, 0, 0), noseDrop));
}
```

### Aircraft Stats Schema

```javascript
// data/aircraft.js

export const AIRCRAFT = {
  F22_RAPTOR: {
    id: 'f22',
    name: 'F-22 Raptor',
    class: 'FIGHTER',
    
    // Performance
    maxSpeed: 2410,        // km/h
    stallSpeed: 300,       // km/h (higher due to thrust vectoring)
    maxThrust: 180,        // Arbitrary units
    dragCoeff: 0.015,
    
    // Handling (radians/second at max input)
    pitchSpeed: 1.8,
    yawSpeed: 0.9,
    rollSpeed: 3.5,
    
    // Combat
    hp: 100,
    armor: 0.8,            // Damage multiplier (lower = tougher)
    
    // Weapons (indices into WEAPONS array)
    standardMissile: 'STD_MSL',
    specialWeapon: 'QAAM',
    specialAmmo: 14,
    gunAmmo: 800,
    
    // PSM capability
    canPSM: true,
    psmSpeedRange: [280, 450], // km/h
    
    // Visual
    wireframeColor: 0x00ff88,
    accentColor: 0xff4444,
  },
  
  A10_WARTHOG: {
    id: 'a10',
    name: 'A-10C Thunderbolt II',
    class: 'ATTACKER',
    
    maxSpeed: 706,
    stallSpeed: 180,
    maxThrust: 80,
    dragCoeff: 0.025,
    
    pitchSpeed: 1.2,
    yawSpeed: 0.7,
    rollSpeed: 2.0,
    
    hp: 200,
    armor: 0.5,
    
    standardMissile: 'STD_MSL',
    specialWeapon: 'UGB',
    specialAmmo: 24,
    gunAmmo: 1200,
    
    canPSM: false,
    
    wireframeColor: 0x88ff00,
    accentColor: 0xffaa00,
  },
  
  // Additional aircraft...
};
```

---

## Targeting and Lock-On System

### Lock-On Mechanics

Targets must satisfy two conditions for lock-on:
1. Within **lock-on range** (weapon-dependent)
2. Within **lock-on cone** (HUD boresight area)

```javascript
// TargetingSystem.jsx

const LOCK_CONE_ANGLE = 30; // degrees from center
const LOCK_CONE_COS = Math.cos(LOCK_CONE_ANGLE * Math.PI / 180);

function canLockTarget(playerPos, playerForward, targetPos, weaponRange) {
  const toTarget = targetPos.clone().sub(playerPos);
  const distance = toTarget.length();
  
  // Range check
  if (distance > weaponRange) return false;
  
  // Cone check
  toTarget.normalize();
  const dot = playerForward.dot(toTarget);
  if (dot < LOCK_CONE_COS) return false;
  
  return true;
}
```

### Lock-On State Machine

```
SEARCHING → ACQUIRING → LOCKED → FIRED
    ↑           ↓         ↓
    └───────────┴─────────┘ (target lost)
```

```javascript
// State definitions
const LOCK_STATE = {
  SEARCHING: 'searching',   // No valid target
  ACQUIRING: 'acquiring',   // Target in cone, building lock
  LOCKED: 'locked',         // Ready to fire
  FIRED: 'fired',           // Missile in flight to this target
};

// Lock acquisition timing
const LOCK_TIME = {
  STD_MSL: 0.8,   // seconds to acquire
  QAAM: 1.2,
  '4AAM': 0.5,    // Faster per-target, but needs multiple
  '6AAM': 0.4,
  '8AAM': 0.35,
};
```

### Multi-Lock (Swarm Missiles)

Multi-lock weapons (4AAM, 6AAM, 8AAM) acquire targets sequentially. Player decides when to fire—early for fewer missiles or wait for maximum saturation.

```javascript
// useTargeting.js hook

function useMultiLock(weaponType, maxLocks) {
  const [lockedTargets, setLockedTargets] = useState([]);
  const [currentLockProgress, setCurrentLockProgress] = useState(0);
  const targets = useTargetStore(s => s.enemies);
  const playerState = usePlayerStore(s => s.state);
  
  useFrame((_, delta) => {
    if (lockedTargets.length >= maxLocks) return;
    
    // Find next lockable target (not already locked)
    const lockableTargets = targets.filter(t => 
      canLockTarget(playerState.position, playerState.forward, t.position, WEAPONS[weaponType].range) &&
      !lockedTargets.includes(t.id)
    );
    
    if (lockableTargets.length === 0) {
      setCurrentLockProgress(0);
      return;
    }
    
    // Progress lock on nearest eligible target
    const nearest = lockableTargets.sort((a, b) => 
      a.position.distanceTo(playerState.position) - 
      b.position.distanceTo(playerState.position)
    )[0];
    
    setCurrentLockProgress(prev => {
      const next = prev + delta / LOCK_TIME[weaponType];
      if (next >= 1) {
        setLockedTargets(prev => [...prev, nearest.id]);
        return 0;
      }
      return next;
    });
  });
  
  const fire = useCallback(() => {
    if (lockedTargets.length === 0) return;
    
    // Fire missiles to all locked targets
    lockedTargets.forEach((targetId, index) => {
      // Stagger launch slightly for visual effect
      setTimeout(() => {
        spawnMissile(weaponType, targetId);
      }, index * 100);
    });
    
    setLockedTargets([]);
  }, [lockedTargets, weaponType]);
  
  return { lockedTargets, currentLockProgress, fire };
}
```

### Dynamic Launch Zone (DLZ) Visualization

The DLZ bar communicates weapon range without cluttering the HUD.

```javascript
// hud/DLZBar.jsx

function DLZBar({ targetDistance, weapon }) {
  const { minRange, optimalMin, optimalMax, maxRange } = WEAPONS[weapon].dlz;
  
  // Calculate caret position (0-1 from bottom to top)
  const caretPos = 1 - (targetDistance / maxRange);
  
  // Calculate optimal zone (thick section)
  const optimalStart = 1 - (optimalMax / maxRange);
  const optimalEnd = 1 - (optimalMin / maxRange);
  
  const inRange = targetDistance <= maxRange && targetDistance >= minRange;
  const inOptimal = targetDistance <= optimalMax && targetDistance >= optimalMin;
  
  return (
    <div className="dlz-bar">
      <div className="dlz-track">
        {/* Optimal zone (thick section) */}
        <div 
          className="dlz-optimal"
          style={{
            bottom: `${optimalStart * 100}%`,
            height: `${(optimalEnd - optimalStart) * 100}%`,
          }}
        />
        
        {/* Caret (target distance indicator) */}
        <div 
          className={`dlz-caret ${inOptimal ? 'optimal' : inRange ? 'valid' : 'invalid'}`}
          style={{ bottom: `${caretPos * 100}%` }}
        />
      </div>
      
      {/* Range numbers */}
      <span className="dlz-max">{maxRange}</span>
      <span className="dlz-min">{minRange}</span>
    </div>
  );
}
```

---

## Missile Guidance System

Missiles use **proportional navigation**—steering toward the predicted intercept point rather than directly at the target.

```javascript
// MissileSystem.jsx

const PN_CONSTANT = 3; // Navigation constant (higher = more aggressive turns)

function updateMissile(missile, target, deltaTime) {
  const { position, velocity, stats } = missile;
  const speed = velocity.length();
  
  // Line-of-sight rate
  const los = target.position.clone().sub(position);
  const losDistance = los.length();
  los.normalize();
  
  // Closing velocity
  const closingVel = velocity.clone().sub(target.velocity);
  const closingSpeed = closingVel.dot(los);
  
  // Calculate lead angle
  const leadTime = losDistance / (closingSpeed + speed);
  const predictedPos = target.position.clone()
    .addScaledVector(target.velocity, leadTime * 0.5);
  
  // Desired heading
  const desiredDir = predictedPos.sub(position).normalize();
  
  // Current heading
  const currentDir = velocity.clone().normalize();
  
  // Turn toward intercept (limited by turn rate)
  const maxTurn = stats.turnRate * deltaTime;
  const turnAngle = Math.acos(Math.min(1, currentDir.dot(desiredDir)));
  const actualTurn = Math.min(turnAngle, maxTurn);
  
  if (actualTurn > 0.001) {
    const turnAxis = currentDir.clone().cross(desiredDir).normalize();
    const turnQuat = new Quaternion().setFromAxisAngle(turnAxis, actualTurn);
    velocity.applyQuaternion(turnQuat);
  }
  
  // Maintain speed
  velocity.normalize().multiplyScalar(stats.speed);
  
  // Motor burnout (after fuel time, loses thrust)
  if (missile.flightTime > stats.motorBurnTime) {
    velocity.multiplyScalar(0.998); // Gradual slowdown
  }
  
  // Update position
  position.addScaledVector(velocity, deltaTime);
  missile.flightTime += deltaTime;
  
  // Check for hit
  if (losDistance < stats.proximityFuse) {
    return { hit: true, target: target.id };
  }
  
  // Check for timeout
  if (missile.flightTime > stats.maxFlightTime) {
    return { expired: true };
  }
  
  return { active: true };
}
```

### Missile Stats

```javascript
// data/weapons.js

export const WEAPONS = {
  STD_MSL: {
    id: 'STD_MSL',
    name: 'Standard Missile',
    type: 'missile',
    lockType: 'single',
    
    speed: 1200,           // m/s
    turnRate: 2.5,         // radians/second
    motorBurnTime: 3,      // seconds
    maxFlightTime: 8,      // seconds
    proximityFuse: 15,     // meters
    damage: 100,
    
    dlz: {
      minRange: 200,
      optimalMin: 800,
      optimalMax: 4000,
      maxRange: 5200,
    },
    
    ammo: 80,
  },
  
  QAAM: {
    id: 'QAAM',
    name: 'Quick Maneuver AAM',
    type: 'missile',
    lockType: 'single',
    
    speed: 1400,
    turnRate: 4.5,         // Much higher turn rate
    motorBurnTime: 5,
    maxFlightTime: 12,
    proximityFuse: 20,
    damage: 120,
    reacquire: true,       // Can re-lock if it misses
    
    dlz: {
      minRange: 150,
      optimalMin: 500,
      optimalMax: 3000,
      maxRange: 4500,
    },
    
    ammo: 14,
  },
  
  '4AAM': {
    id: '4AAM',
    name: '4-Target AAM',
    type: 'missile',
    lockType: 'multi',
    maxLocks: 4,
    
    speed: 1100,
    turnRate: 2.0,         // Worse tracking than single-lock
    motorBurnTime: 2.5,
    maxFlightTime: 7,
    proximityFuse: 15,
    damage: 80,
    
    dlz: {
      minRange: 300,
      optimalMin: 1000,
      optimalMax: 4500,
      maxRange: 6000,
    },
    
    ammo: 32,
  },
  
  '6AAM': {
    id: '6AAM',
    name: '6-Target AAM',
    type: 'missile',
    lockType: 'multi',
    maxLocks: 6,
    
    speed: 1000,
    turnRate: 1.8,
    motorBurnTime: 2.5,
    maxFlightTime: 7,
    proximityFuse: 15,
    damage: 70,
    
    dlz: {
      minRange: 400,
      optimalMin: 1200,
      optimalMax: 5000,
      maxRange: 7000,
    },
    
    ammo: 24,
  },
  
  '8AAM': {
    id: '8AAM',
    name: '8-Target AAM',
    type: 'missile',
    lockType: 'multi',
    maxLocks: 8,
    salvoOnly: true,       // Must fire all 8 or none
    
    speed: 950,
    turnRate: 1.6,
    motorBurnTime: 2,
    maxFlightTime: 6,
    proximityFuse: 12,
    damage: 60,
    
    dlz: {
      minRange: 500,
      optimalMin: 1500,
      optimalMax: 5500,
      maxRange: 8000,
    },
    
    ammo: 16,
  },
};
```

### Missile Evasion

Evasion effectiveness depends on angle relative to missile vector, not raw speed.

```javascript
// In FlightSystem.jsx - when missile warning active

function calculateEvasionEffectiveness(playerVelocity, missileDirection) {
  const playerDir = playerVelocity.clone().normalize();
  
  // Cross product magnitude indicates perpendicular movement
  const cross = playerDir.clone().cross(missileDirection);
  const perpendicular = cross.length(); // 0 = parallel, 1 = perpendicular
  
  // Perpendicular movement is most effective
  return perpendicular;
}
```

---

## Wave-Based Survival System

### Wave Structure

Missions consist of escalating waves with point thresholds triggering transitions.

```javascript
// data/waves.js

export const WAVE_CONFIGS = {
  MISSION_01: {
    name: "Carrier Defense",
    timeLimit: 900, // 15 minutes
    parTime: 600,   // 10 minutes for time bonus
    
    waves: [
      {
        id: 1,
        trigger: { type: 'immediate' },
        spawns: [
          { type: 'ENEMY_FIGHTER', count: 4, formation: 'diamond', delay: 0 },
          { type: 'ENEMY_FIGHTER', count: 4, formation: 'line', delay: 15 },
        ],
        objective: "Intercept incoming fighters",
        pointsRequired: 4000, // To advance
      },
      {
        id: 2,
        trigger: { type: 'points', threshold: 4000 },
        spawns: [
          { type: 'ENEMY_BOMBER', count: 2, formation: 'pair', delay: 0 },
          { type: 'ENEMY_FIGHTER', count: 6, formation: 'spread', delay: 10 },
        ],
        objective: "Destroy bombers before they reach the carrier",
        specialSpawn: {
          condition: { type: 'time', underSeconds: 180 },
          type: 'NAMED_ACE',
          id: 'YELLOW_13',
        },
        pointsRequired: 8000,
      },
      {
        id: 3,
        trigger: { type: 'points', threshold: 8000 },
        spawns: [
          { type: 'ENEMY_FIGHTER', count: 12, formation: 'swarm', delay: 0 },
          { type: 'DRONE', count: 8, formation: 'cloud', delay: 20 },
        ],
        objective: "Survive the drone swarm",
        pointsRequired: null, // Final wave - survive to timer
      },
    ],
    
    sRankRequirements: {
      points: 15000,
      time: 480, // Under 8 minutes
    },
  },
};
```

### Wave System Component

```javascript
// systems/WaveSystem.jsx

function WaveSystem() {
  const { currentWave, advanceWave, missionConfig } = useMissionStore();
  const { score, time } = useGameStore();
  const spawnEnemy = useTargetStore(s => s.spawnEnemy);
  
  const wave = missionConfig.waves[currentWave];
  const [spawned, setSpawned] = useState(false);
  const [spawnTimers, setSpawnTimers] = useState([]);
  
  // Check wave advancement
  useEffect(() => {
    if (!wave) return; // Mission complete
    
    if (wave.pointsRequired && score >= wave.pointsRequired) {
      advanceWave();
      setSpawned(false);
    }
  }, [score, wave, advanceWave]);
  
  // Spawn enemies for current wave
  useEffect(() => {
    if (spawned || !wave) return;
    
    // Clear previous timers
    spawnTimers.forEach(clearTimeout);
    
    const newTimers = wave.spawns.map(spawn => {
      return setTimeout(() => {
        for (let i = 0; i < spawn.count; i++) {
          const position = calculateFormationPosition(spawn.formation, i, spawn.count);
          spawnEnemy(spawn.type, position);
        }
      }, spawn.delay * 1000);
    });
    
    setSpawnTimers(newTimers);
    setSpawned(true);
    
    return () => newTimers.forEach(clearTimeout);
  }, [wave, spawned]);
  
  // Check for special spawns (named aces)
  useEffect(() => {
    if (!wave?.specialSpawn) return;
    
    const { condition, type, id } = wave.specialSpawn;
    
    if (condition.type === 'time' && time < condition.underSeconds) {
      spawnEnemy(type, calculateAceSpawnPosition(), { aceId: id });
    }
  }, [wave, time]);
  
  return null; // System component, no render
}
```

### Scoring System

```javascript
// systems/ScoringSystem.jsx

const SCORE_VALUES = {
  ENEMY_FIGHTER: 500,
  ENEMY_BOMBER: 1000,
  DRONE: 300,
  NAMED_ACE: 2000, // Double points
  GROUND_TARGET: 200,
};

const TIME_BONUS_DECAY = 30; // Points per second after par time

function ScoringSystem() {
  const { score, setScore, time, missionConfig } = useGameStore();
  const enemies = useTargetStore(s => s.enemies);
  const previousEnemies = useRef(enemies);
  
  // Detect destroyed enemies
  useEffect(() => {
    const destroyed = previousEnemies.current.filter(
      prev => !enemies.find(curr => curr.id === prev.id)
    );
    
    destroyed.forEach(enemy => {
      const baseScore = SCORE_VALUES[enemy.type] || 100;
      const multiplier = enemy.isAce ? 2 : 1;
      setScore(s => s + baseScore * multiplier);
    });
    
    previousEnemies.current = enemies;
  }, [enemies]);
  
  // Calculate final rank
  const calculateRank = useCallback(() => {
    const { sRankRequirements, parTime } = missionConfig;
    
    // Time bonus (decays after par)
    const overtime = Math.max(0, time - parTime);
    const timeBonus = Math.max(0, 5000 - overtime * TIME_BONUS_DECAY);
    const finalScore = score + timeBonus;
    
    if (finalScore >= sRankRequirements.points && time <= sRankRequirements.time) {
      return 'S';
    } else if (finalScore >= sRankRequirements.points * 0.8) {
      return 'A';
    } else if (finalScore >= sRankRequirements.points * 0.6) {
      return 'B';
    } else if (finalScore >= sRankRequirements.points * 0.4) {
      return 'C';
    } else {
      return 'D';
    }
  }, [score, time, missionConfig]);
  
  return null;
}
```

---

## Flying Mega-Carrier (Player Base)

The carrier serves as mission hub, hangar, and in-mission resupply point.

### Carrier Component

```javascript
// entities/MegaCarrier.jsx

function MegaCarrier({ position = [0, 8000, 0] }) {
  const carrierRef = useRef();
  const { carrierState } = useMissionStore();
  
  // Carrier slowly orbits
  useFrame((_, delta) => {
    if (!carrierRef.current) return;
    carrierRef.current.rotation.y += delta * 0.01;
  });
  
  return (
    <group ref={carrierRef} position={position}>
      {/* Main hull - elongated hexagonal prism */}
      <WireframeMesh
        geometry="carrierHull"
        color={0x00ffaa}
        scale={[200, 30, 80]}
      />
      
      {/* Flight deck (top surface) */}
      <WireframeMesh
        geometry="flightDeck"
        position={[0, 20, 0]}
        color={0x00ffaa}
        scale={[180, 2, 60]}
      />
      
      {/* Hangar bay (internal) */}
      <HangarBay position={[0, 0, -20]} />
      
      {/* Engine pods (rear) */}
      {[-30, 0, 30].map((x, i) => (
        <EnginePod key={i} position={[x, -10, 40]} />
      ))}
      
      {/* Defense turrets */}
      {TURRET_POSITIONS.map((pos, i) => (
        <DefenseTurret key={i} position={pos} />
      ))}
      
      {/* Shield generator (when active) */}
      {carrierState.shieldActive && (
        <ShieldBubble radius={250} color={0x4444ff} />
      )}
      
      {/* Return line indicator */}
      <ReturnLineZone position={[0, 0, -100]} radius={150} />
    </group>
  );
}
```

### Return Line (Resupply Zone)

Entering the return zone triggers resupply sequence.

```javascript
// entities/ReturnLineZone.jsx

function ReturnLineZone({ position, radius }) {
  const playerPos = usePlayerStore(s => s.position);
  const { resupply, isResupplying } = usePlayerStore();
  
  useFrame(() => {
    const distance = new Vector3(...position).distanceTo(playerPos);
    
    if (distance < radius && !isResupplying) {
      resupply();
    }
  });
  
  return (
    <group position={position}>
      {/* Dashed circle indicating zone */}
      <line>
        <bufferGeometry>
          <bufferAttribute
            attach="attributes-position"
            count={64}
            array={generateCirclePoints(radius, 64)}
            itemSize={3}
          />
        </bufferGeometry>
        <lineDashedMaterial color={0x00ff00} dashSize={10} gapSize={10} />
      </line>
      
      {/* "RTB" text */}
      <Text
        position={[0, 50, 0]}
        fontSize={20}
        color={0x00ff00}
        anchorX="center"
      >
        RTB ZONE
      </Text>
    </group>
  );
}
```

### Resupply Logic

```javascript
// In usePlayerStore.js

resupply: () => {
  set({ isResupplying: true });
  
  // Resupply takes 5 seconds
  const interval = setInterval(() => {
    set(state => ({
      missiles: Math.min(state.missiles + 10, state.maxMissiles),
      specialAmmo: Math.min(state.specialAmmo + 2, state.maxSpecialAmmo),
      hp: Math.min(state.hp + 20, state.maxHp),
    }));
  }, 1000);
  
  setTimeout(() => {
    clearInterval(interval);
    set({ isResupplying: false });
  }, 5000);
},
```

---

## Visual Style: Wireframe with SVGA Vertex Shading

### Wireframe Shader

```glsl
// shaders/wireframe.vert

varying vec3 vPosition;
varying vec3 vNormal;
varying vec2 vBarycentricCoord;

void main() {
  vPosition = position;
  vNormal = normal;
  vBarycentricCoord = uv; // Repurpose UV for edge detection
  
  gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
}
```

```glsl
// shaders/wireframe.frag

uniform vec3 uColor;
uniform float uEdgeWidth;
uniform float uGlowIntensity;
uniform float uScanlineIntensity;

varying vec3 vPosition;
varying vec3 vNormal;
varying vec2 vBarycentricCoord;

float edgeFactor() {
  vec3 bary = vec3(vBarycentricCoord, 1.0 - vBarycentricCoord.x - vBarycentricCoord.y);
  vec3 d = fwidth(bary);
  vec3 f = step(d * uEdgeWidth, bary);
  return min(min(f.x, f.y), f.z);
}

void main() {
  float edge = 1.0 - edgeFactor();
  
  // Scanline effect (SVGA style)
  float scanline = sin(gl_FragCoord.y * 3.14159 * 2.0) * 0.5 + 0.5;
  scanline = mix(1.0, scanline, uScanlineIntensity);
  
  // Vertex shading (color varies by normal)
  float shade = dot(normalize(vNormal), vec3(0.0, 1.0, 0.5)) * 0.5 + 0.5;
  
  // Final color
  vec3 color = uColor * shade * scanline;
  
  // Edge glow
  color += uColor * edge * uGlowIntensity;
  
  // Discard fill (wireframe only)
  if (edge < 0.1) discard;
  
  gl_FragColor = vec4(color, edge);
}
```

### WireframeMesh Component

```javascript
// components/WireframeMesh.jsx

function WireframeMesh({ 
  geometry, 
  color = 0x00ff88, 
  edgeWidth = 1.5,
  glowIntensity = 2.0,
  scanlineIntensity = 0.1,
  ...props 
}) {
  const materialRef = useRef();
  
  // Animate glow
  useFrame(({ clock }) => {
    if (materialRef.current) {
      materialRef.current.uniforms.uGlowIntensity.value = 
        glowIntensity + Math.sin(clock.elapsedTime * 2) * 0.3;
    }
  });
  
  return (
    <mesh {...props}>
      <primitive object={GEOMETRIES[geometry]} attach="geometry" />
      <shaderMaterial
        ref={materialRef}
        vertexShader={wireframeVert}
        fragmentShader={wireframeFrag}
        uniforms={{
          uColor: { value: new Color(color) },
          uEdgeWidth: { value: edgeWidth },
          uGlowIntensity: { value: glowIntensity },
          uScanlineIntensity: { value: scanlineIntensity },
        }}
        transparent
        side={DoubleSide}
      />
    </mesh>
  );
}
```

### Post-Processing Effects

```javascript
// components/game/PostProcessing.jsx

import { EffectComposer, Bloom, Scanline, ChromaticAberration } from '@react-three/postprocessing';

function PostProcessing() {
  return (
    <EffectComposer>
      {/* Bloom for glow effect */}
      <Bloom
        intensity={1.5}
        luminanceThreshold={0.2}
        luminanceSmoothing={0.9}
        radius={0.8}
      />
      
      {/* CRT-style scanlines */}
      <Scanline density={1.5} opacity={0.05} />
      
      {/* Subtle chromatic aberration for retro feel */}
      <ChromaticAberration offset={[0.0005, 0.0005]} />
    </EffectComposer>
  );
}
```

### Explosion Effect

```javascript
// entities/Explosion.jsx

function Explosion({ position, color = 0xff4400, size = 20 }) {
  const pointsRef = useRef();
  const [particles] = useState(() => {
    const count = 50;
    const positions = new Float32Array(count * 3);
    const velocities = [];
    
    for (let i = 0; i < count; i++) {
      positions[i * 3] = 0;
      positions[i * 3 + 1] = 0;
      positions[i * 3 + 2] = 0;
      
      // Random outward velocity
      velocities.push(new Vector3(
        (Math.random() - 0.5) * 2,
        (Math.random() - 0.5) * 2,
        (Math.random() - 0.5) * 2
      ).normalize().multiplyScalar(50 + Math.random() * 50));
    }
    
    return { positions, velocities };
  });
  
  const [life, setLife] = useState(1);
  
  useFrame((_, delta) => {
    if (!pointsRef.current) return;
    
    const positions = pointsRef.current.geometry.attributes.position.array;
    
    for (let i = 0; i < particles.velocities.length; i++) {
      positions[i * 3] += particles.velocities[i].x * delta;
      positions[i * 3 + 1] += particles.velocities[i].y * delta;
      positions[i * 3 + 2] += particles.velocities[i].z * delta;
      
      // Slow down
      particles.velocities[i].multiplyScalar(0.95);
    }
    
    pointsRef.current.geometry.attributes.position.needsUpdate = true;
    
    setLife(l => l - delta * 0.5);
  });
  
  if (life <= 0) return null;
  
  return (
    <points ref={pointsRef} position={position}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          count={particles.positions.length / 3}
          array={particles.positions}
          itemSize={3}
        />
      </bufferGeometry>
      <pointsMaterial
        color={color}
        size={size * life}
        transparent
        opacity={life}
        sizeAttenuation
      />
    </points>
  );
}
```

---

## HUD System

The HUD uses HTML/CSS overlay for crisp 2D elements, positioned absolutely over the Canvas.

### HUD Container

```javascript
// hud/HUD.jsx

function HUD() {
  const gameState = useGameStore(s => s.state);
  
  if (gameState !== 'playing') return null;
  
  return (
    <div className="hud-container">
      {/* Left side */}
      <div className="hud-left">
        <Speedometer />
        <ThrottleIndicator />
      </div>
      
      {/* Right side */}
      <div className="hud-right">
        <Altimeter />
        <VerticalSpeedIndicator />
      </div>
      
      {/* Top center */}
      <div className="hud-top">
        <Compass />
        <MissionTimer />
      </div>
      
      {/* Bottom left */}
      <div className="hud-bottom-left">
        <Radar />
      </div>
      
      {/* Bottom right */}
      <div className="hud-bottom-right">
        <WeaponDisplay />
        <AmmoCounter />
      </div>
      
      {/* Center (dynamic) */}
      <div className="hud-center">
        <Crosshair />
        <TargetBoxes />
        <MissileWarning />
        <LockIndicator />
      </div>
      
      {/* Wave info */}
      <div className="hud-wave">
        <WaveIndicator />
        <ScoreDisplay />
      </div>
    </div>
  );
}
```

### HUD Styles

```css
/* styles/hud.css */

.hud-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  font-family: 'Share Tech Mono', monospace;
  color: #00ff88;
  text-shadow: 0 0 10px #00ff88;
}

.hud-left {
  position: absolute;
  left: 40px;
  top: 50%;
  transform: translateY(-50%);
}

.hud-right {
  position: absolute;
  right: 40px;
  top: 50%;
  transform: translateY(-50%);
}

.speedometer {
  font-size: 32px;
  letter-spacing: 2px;
}

.speedometer-value {
  font-size: 48px;
  font-weight: bold;
}

.speedometer-unit {
  font-size: 16px;
  opacity: 0.7;
}

.radar {
  width: 150px;
  height: 150px;
  border: 2px solid #00ff88;
  border-radius: 50%;
  position: relative;
  background: rgba(0, 255, 136, 0.05);
}

.radar-sweep {
  position: absolute;
  width: 50%;
  height: 2px;
  background: linear-gradient(to right, transparent, #00ff88);
  top: 50%;
  left: 50%;
  transform-origin: left center;
  animation: radar-sweep 2s linear infinite;
}

@keyframes radar-sweep {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.radar-blip {
  position: absolute;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.radar-blip.enemy { background: #ff4444; }
.radar-blip.friendly { background: #4444ff; }
.radar-blip.target { background: #ffff00; }

.missile-warning {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 24px;
  color: #ff0000;
  animation: warning-flash 0.2s ease-in-out infinite;
}

@keyframes warning-flash {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}

.target-box {
  position: absolute;
  border: 2px solid currentColor;
  pointer-events: none;
}

.target-box.searching { border-style: dashed; color: #888888; }
.target-box.acquiring { border-style: solid; color: #ffff00; animation: acquiring-pulse 0.3s infinite; }
.target-box.locked { border-style: solid; color: #ff0000; }

@keyframes acquiring-pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}
```

### Target Box Rendering

```javascript
// hud/TargetBoxes.jsx

function TargetBoxes() {
  const enemies = useTargetStore(s => s.enemies);
  const { position: playerPos, forward } = usePlayerStore();
  const lockState = useTargetStore(s => s.lockState);
  const camera = useThree(s => s.camera);
  
  return (
    <>
      {enemies.map(enemy => {
        // Project 3D position to screen
        const screenPos = enemy.position.clone().project(camera);
        
        // Check if in front of camera
        if (screenPos.z > 1) return null;
        
        // Convert to pixel coordinates
        const x = (screenPos.x * 0.5 + 0.5) * window.innerWidth;
        const y = (1 - (screenPos.y * 0.5 + 0.5)) * window.innerHeight;
        
        // Determine lock state
        const state = lockState[enemy.id] || 'searching';
        
        // Calculate box size based on distance
        const distance = enemy.position.distanceTo(playerPos);
        const size = Math.max(20, 100 - distance / 50);
        
        return (
          <div
            key={enemy.id}
            className={`target-box ${state}`}
            style={{
              left: x - size / 2,
              top: y - size / 2,
              width: size,
              height: size,
            }}
          >
            {/* Distance readout */}
            <span className="target-distance">
              {Math.round(distance)}m
            </span>
            
            {/* Target type */}
            <span className="target-type">
              {enemy.type}
            </span>
            
            {/* Lock diamond (when locked) */}
            {state === 'locked' && (
              <div className="lock-diamond">◆</div>
            )}
          </div>
        );
      })}
    </>
  );
}
```

---

## Audio System

### Audio Manager

```javascript
// core/AudioManager.js

import { Howl, Howler } from 'howler';

class AudioManager {
  constructor() {
    this.sounds = {};
    this.music = null;
    this.missileWarningLoop = null;
  }
  
  loadSounds() {
    this.sounds = {
      // Weapons
      missilefire: new Howl({ src: ['/audio/missile_fire.wav'], volume: 0.7 }),
      missilehit: new Howl({ src: ['/audio/missile_hit.wav'], volume: 0.8 }),
      gunfire: new Howl({ src: ['/audio/gun_burst.wav'], volume: 0.5 }),
      
      // Alerts
      lockon: new Howl({ src: ['/audio/lockon.wav'], volume: 0.6 }),
      lockconfirm: new Howl({ src: ['/audio/lock_confirm.wav'], volume: 0.8 }),
      missilewarning: new Howl({ 
        src: ['/audio/missile_warning.wav'], 
        volume: 1.0,
        loop: true,
      }),
      
      // Voice
      missile_missile: new Howl({ src: ['/audio/voice_missile.wav'], volume: 0.9 }),
      fox_two: new Howl({ src: ['/audio/voice_fox2.wav'], volume: 0.7 }),
      target_destroyed: new Howl({ src: ['/audio/voice_splash.wav'], volume: 0.7 }),
      
      // Feedback
      hit: new Howl({ src: ['/audio/hit.wav'], volume: 0.8 }),
      explosion: new Howl({ src: ['/audio/explosion.wav'], volume: 0.9 }),
      
      // UI
      select: new Howl({ src: ['/audio/ui_select.wav'], volume: 0.5 }),
      confirm: new Howl({ src: ['/audio/ui_confirm.wav'], volume: 0.5 }),
    };
  }
  
  play(soundId, options = {}) {
    const sound = this.sounds[soundId];
    if (!sound) return;
    
    if (options.rate) sound.rate(options.rate);
    if (options.volume) sound.volume(options.volume);
    
    return sound.play();
  }
  
  startMissileWarning(distance) {
    if (this.missileWarningLoop) return;
    
    // Beep rate increases as missile gets closer
    const baseRate = 0.5;
    const maxRate = 3.0;
    const rate = baseRate + (1 - distance / 5000) * (maxRate - baseRate);
    
    this.missileWarningLoop = setInterval(() => {
      this.sounds.missilewarning.play();
    }, 1000 / rate);
    
    // Also play voice warning
    this.play('missile_missile');
  }
  
  stopMissileWarning() {
    if (this.missileWarningLoop) {
      clearInterval(this.missileWarningLoop);
      this.missileWarningLoop = null;
    }
  }
  
  updateMissileWarningRate(distance) {
    if (!this.missileWarningLoop) return;
    
    this.stopMissileWarning();
    this.startMissileWarning(distance);
  }
  
  playMusic(trackId, fadeIn = true) {
    if (this.music) {
      this.music.fade(1, 0, 1000);
      this.music.once('fade', () => this.music.stop());
    }
    
    this.music = new Howl({
      src: [`/audio/music/${trackId}.mp3`],
      volume: fadeIn ? 0 : 0.6,
      loop: true,
    });
    
    this.music.play();
    
    if (fadeIn) {
      this.music.fade(0, 0.6, 2000);
    }
  }
  
  setMusicIntensity(intensity) {
    // Crossfade between calm and intense layers
    // Implementation depends on music format
  }
}

export const audioManager = new AudioManager();
```

### Audio Hook

```javascript
// hooks/useAudio.js

function useAudio() {
  const play = useCallback((soundId, options) => {
    audioManager.play(soundId, options);
  }, []);
  
  const playMusic = useCallback((trackId) => {
    audioManager.playMusic(trackId);
  }, []);
  
  return { play, playMusic };
}
```

---

## Input System

### Input Manager

```javascript
// core/InputManager.js

class InputManager {
  constructor() {
    this.keys = {};
    this.gamepad = null;
    this.callbacks = new Set();
    
    this.setupKeyboard();
    this.setupGamepad();
  }
  
  setupKeyboard() {
    window.addEventListener('keydown', (e) => {
      this.keys[e.code] = true;
      this.notify();
    });
    
    window.addEventListener('keyup', (e) => {
      this.keys[e.code] = false;
      this.notify();
    });
  }
  
  setupGamepad() {
    window.addEventListener('gamepadconnected', (e) => {
      this.gamepad = e.gamepad;
    });
    
    window.addEventListener('gamepaddisconnected', () => {
      this.gamepad = null;
    });
  }
  
  getInput() {
    // Poll gamepad
    if (this.gamepad) {
      const gp = navigator.getGamepads()[this.gamepad.index];
      if (gp) {
        return {
          pitch: -gp.axes[1],         // Left stick Y (inverted)
          roll: gp.axes[0],           // Left stick X
          yaw: gp.axes[2],            // Right stick X
          throttle: this.getThrottle(gp),
          accelerate: gp.buttons[7].pressed,  // RT
          decelerate: gp.buttons[6].pressed,  // LT
          fire: gp.buttons[0].pressed,        // A
          switchWeapon: gp.buttons[2].pressed, // X
          flare: gp.buttons[1].pressed,       // B
        };
      }
    }
    
    // Keyboard fallback
    return {
      pitch: (this.keys['KeyS'] ? 1 : 0) - (this.keys['KeyW'] ? 1 : 0),
      roll: (this.keys['KeyD'] ? 1 : 0) - (this.keys['KeyA'] ? 1 : 0),
      yaw: (this.keys['KeyE'] ? 1 : 0) - (this.keys['KeyQ'] ? 1 : 0),
      throttle: this.keys['ShiftLeft'] ? 1 : this.keys['ControlLeft'] ? -1 : 0,
      accelerate: this.keys['ShiftLeft'] || false,
      decelerate: this.keys['ControlLeft'] || false,
      fire: this.keys['Space'] || false,
      switchWeapon: this.keys['Tab'] || false,
      flare: this.keys['KeyF'] || false,
    };
  }
  
  getThrottle(gp) {
    // Use triggers for throttle
    const accel = gp.buttons[7].value;
    const decel = gp.buttons[6].value;
    return accel - decel;
  }
  
  subscribe(callback) {
    this.callbacks.add(callback);
    return () => this.callbacks.delete(callback);
  }
  
  notify() {
    const input = this.getInput();
    this.callbacks.forEach(cb => cb(input));
  }
}

export const inputManager = new InputManager();
```

### Input Hook

```javascript
// hooks/useInput.js

function useInput() {
  const [input, setInput] = useState(inputManager.getInput());
  
  useEffect(() => {
    const unsubscribe = inputManager.subscribe(setInput);
    
    // Poll gamepad every frame
    let rafId;
    const poll = () => {
      setInput(inputManager.getInput());
      rafId = requestAnimationFrame(poll);
    };
    poll();
    
    return () => {
      unsubscribe();
      cancelAnimationFrame(rafId);
    };
  }, []);
  
  return input;
}
```

---

## State Management

### Game Store

```javascript
// state/useGameStore.js

import { create } from 'zustand';

export const useGameStore = create((set, get) => ({
  // Game state
  state: 'menu', // 'menu' | 'briefing' | 'playing' | 'paused' | 'results'
  
  // Mission data
  score: 0,
  time: 0,
  missionId: null,
  
  // Methods
  setState: (state) => set({ state }),
  
  setScore: (fn) => set(state => ({ 
    score: typeof fn === 'function' ? fn(state.score) : fn 
  })),
  
  tick: (delta) => set(state => ({ time: state.time + delta })),
  
  startMission: (missionId) => set({
    state: 'playing',
    missionId,
    score: 0,
    time: 0,
  }),
  
  endMission: () => set({ state: 'results' }),
  
  reset: () => set({
    state: 'menu',
    score: 0,
    time: 0,
    missionId: null,
  }),
}));
```

### Player Store

```javascript
// state/usePlayerStore.js

import { create } from 'zustand';
import { Vector3, Quaternion } from 'three';

export const usePlayerStore = create((set, get) => ({
  // Aircraft selection
  aircraftId: 'f22',
  specialWeapon: 'QAAM',
  
  // Transform
  position: new Vector3(0, 5000, 0),
  velocity: new Vector3(0, 0, -200),
  rotation: new Quaternion(),
  
  // Derived
  speed: 200,
  altitude: 5000,
  heading: 0,
  
  // Combat
  hp: 100,
  maxHp: 100,
  missiles: 80,
  maxMissiles: 80,
  specialAmmo: 14,
  maxSpecialAmmo: 14,
  gunAmmo: 800,
  flares: 3,
  
  // State flags
  isResupplying: false,
  isDamaged: false,
  
  // Methods
  updateTransform: (position, velocity, rotation) => {
    const speed = velocity.length() * 3.6; // m/s to km/h
    const altitude = position.y;
    const heading = Math.atan2(velocity.x, -velocity.z) * 180 / Math.PI;
    
    set({ position, velocity, rotation, speed, altitude, heading });
  },
  
  takeDamage: (amount) => {
    const { hp } = get();
    const newHp = Math.max(0, hp - amount);
    set({ hp: newHp, isDamaged: true });
    
    setTimeout(() => set({ isDamaged: false }), 500);
    
    if (newHp <= 0) {
      // Player destroyed
      useGameStore.getState().endMission();
    }
  },
  
  fireMissile: (type) => {
    const state = get();
    if (type === 'standard' && state.missiles > 0) {
      set({ missiles: state.missiles - 1 });
      return true;
    }
    if (type === 'special' && state.specialAmmo > 0) {
      set({ specialAmmo: state.specialAmmo - 1 });
      return true;
    }
    return false;
  },
  
  selectAircraft: (id) => {
    const aircraft = AIRCRAFT[id];
    set({
      aircraftId: id,
      specialWeapon: aircraft.specialWeapon,
      maxHp: aircraft.hp,
      hp: aircraft.hp,
      maxMissiles: WEAPONS.STD_MSL.ammo,
      missiles: WEAPONS.STD_MSL.ammo,
      maxSpecialAmmo: aircraft.specialAmmo,
      specialAmmo: aircraft.specialAmmo,
      gunAmmo: aircraft.gunAmmo,
    });
  },
  
  resupply: () => {
    set({ isResupplying: true });
    
    const interval = setInterval(() => {
      set(state => ({
        missiles: Math.min(state.missiles + 10, state.maxMissiles),
        specialAmmo: Math.min(state.specialAmmo + 2, state.maxSpecialAmmo),
        hp: Math.min(state.hp + 20, state.maxHp),
        flares: 3,
      }));
    }, 1000);
    
    setTimeout(() => {
      clearInterval(interval);
      set({ isResupplying: false });
    }, 5000);
  },
}));
```

### Target Store

```javascript
// state/useTargetStore.js

import { create } from 'zustand';
import { Vector3 } from 'three';

let entityId = 0;

export const useTargetStore = create((set, get) => ({
  enemies: [],
  missiles: [],
  explosions: [],
  
  lockState: {}, // { [entityId]: 'searching' | 'acquiring' | 'locked' }
  lockedTargets: [], // For multi-lock weapons
  
  spawnEnemy: (type, position, options = {}) => {
    const id = ++entityId;
    const enemy = {
      id,
      type,
      position: position.clone(),
      velocity: new Vector3(0, 0, 0),
      rotation: new Quaternion(),
      hp: ENEMY_STATS[type].hp,
      isAce: options.aceId != null,
      aceId: options.aceId,
    };
    
    set(state => ({ enemies: [...state.enemies, enemy] }));
    return id;
  },
  
  updateEnemy: (id, updates) => {
    set(state => ({
      enemies: state.enemies.map(e => 
        e.id === id ? { ...e, ...updates } : e
      ),
    }));
  },
  
  destroyEnemy: (id) => {
    const enemy = get().enemies.find(e => e.id === id);
    if (enemy) {
      // Spawn explosion
      get().spawnExplosion(enemy.position.clone());
      
      // Remove enemy
      set(state => ({
        enemies: state.enemies.filter(e => e.id !== id),
        lockState: Object.fromEntries(
          Object.entries(state.lockState).filter(([k]) => k !== String(id))
        ),
      }));
    }
  },
  
  spawnMissile: (type, targetId, origin) => {
    const id = ++entityId;
    const missile = {
      id,
      type,
      targetId,
      position: origin.clone(),
      velocity: new Vector3(0, 0, -WEAPONS[type].speed),
      flightTime: 0,
    };
    
    set(state => ({ missiles: [...state.missiles, missile] }));
    return id;
  },
  
  updateMissile: (id, updates) => {
    set(state => ({
      missiles: state.missiles.map(m => 
        m.id === id ? { ...m, ...updates } : m
      ),
    }));
  },
  
  destroyMissile: (id) => {
    set(state => ({
      missiles: state.missiles.filter(m => m.id !== id),
    }));
  },
  
  spawnExplosion: (position, color = 0xff4400) => {
    const id = ++entityId;
    set(state => ({
      explosions: [...state.explosions, { id, position, color, time: 0 }],
    }));
    
    // Auto-remove after 2 seconds
    setTimeout(() => {
      set(state => ({
        explosions: state.explosions.filter(e => e.id !== id),
      }));
    }, 2000);
  },
  
  setLockState: (targetId, state) => {
    set(s => ({
      lockState: { ...s.lockState, [targetId]: state },
    }));
  },
  
  addLockedTarget: (targetId) => {
    set(state => ({
      lockedTargets: [...state.lockedTargets, targetId],
    }));
  },
  
  clearLockedTargets: () => {
    set({ lockedTargets: [] });
  },
}));
```

---

## Testing Scenarios

### Flight System Tests

1. **Stall Recovery**: Reduce throttle to stall, verify nose drops, increase throttle, verify recovery
2. **High-G Turn**: Initiate high-G at 800 km/h, verify speed bleeds to ~400 km/h within 3 seconds
3. **Speed Limits**: Full throttle for 30 seconds, verify max speed matches aircraft stats
4. **Altitude Limits**: Attempt to fly below 50m or above 25000m, verify enforcement

### Targeting Tests

1. **Lock Cone**: Target at 45° off boresight should not lock
2. **Lock Range**: Target beyond weapon range should not lock
3. **Lock Timing**: Standard missile should lock in ~0.8 seconds when conditions met
4. **Multi-Lock Sequence**: 4AAM should lock 4 targets sequentially

### Missile Tests

1. **Tracking**: Fire at maneuvering target, verify proportional navigation
2. **Evasion**: Fly perpendicular to missile, verify higher dodge rate than parallel
3. **Timeout**: Missile should expire after max flight time
4. **QAAM Re-track**: Fire QAAM, dodge, verify it attempts re-acquisition

### Wave System Tests

1. **Wave Trigger**: Reach point threshold, verify next wave spawns
2. **Named Ace Spawn**: Complete wave under time threshold, verify ace appears
3. **Final Wave**: Survive timer, verify mission success

### UI Tests

1. **HUD Elements**: All displays update in real-time
2. **Target Boxes**: Project correctly to screen space
3. **Missile Warning**: Triggers when enemy missile fired, stops when evaded/hit

---

## Performance Goals

| Metric | Target | Method |
|--------|--------|--------|
| Frame Rate | 60 FPS stable | Object pooling, instancing, LOD |
| Draw Calls | <100 per frame | Geometry merging, instanced meshes |
| Memory | <200MB heap | Asset streaming, disposal |
| Load Time | <3 seconds | Code splitting, lazy loading |
| Input Latency | <16ms | Direct RAF polling |

### Optimization Strategies

1. **Object Pooling**: Pre-allocate missiles and explosions, recycle instead of create/destroy
2. **Instanced Meshes**: Use `InstancedMesh` for enemies of same type
3. **Geometry Merging**: Combine static wireframe elements
4. **LOD for Distance**: Reduce wireframe detail for distant objects
5. **Frustum Culling**: R3F handles automatically, ensure large objects have proper bounds
6. **State Updates**: Batch Zustand updates, use selectors to prevent unnecessary re-renders

---

## Extended Features (Optional)

### Multiplayer (Future)

Implement via WebRTC for peer-to-peer or WebSocket for authoritative server. State sync focuses on:
- Player transforms (interpolated)
- Missile spawns
- Damage events

### VR Support (Future)

React Three Fiber supports WebXR natively:
```jsx
<Canvas vr>
  <XR>
    <Controllers />
    <Scene />
  </XR>
</Canvas>
```

Cockpit view with head tracking. Controller-based throttle and targeting.

### Mission Editor (Future)

JSON-based wave definitions already support data-driven missions. Visual editor could:
- Place spawn points
- Define trigger conditions
- Set scoring thresholds
- Preview in-engine

---

## Getting Started

```bash
# Clone repository
git clone https://github.com/your-org/react-ace.git
cd react-ace

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

### Project Structure After Build

```
dist/
├── index.html
├── assets/
│   ├── index.[hash].js
│   ├── vendor.[hash].js
│   └── style.[hash].css
├── audio/
│   ├── *.wav
│   └── music/*.mp3
└── models/
    └── *.json (geometry data)
```

---

## Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@react-three/fiber": "^8.15.0",
    "@react-three/drei": "^9.88.0",
    "@react-three/postprocessing": "^2.15.0",
    "three": "^0.158.0",
    "zustand": "^4.4.0",
    "howler": "^2.2.4"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "typescript": "^5.3.0"
  }
}
```

---

*This README provides sufficient detail for implementation. All core mechanics, data structures, component architecture, and visual specifications are defined. Generate functional code following these specifications exactly.*
