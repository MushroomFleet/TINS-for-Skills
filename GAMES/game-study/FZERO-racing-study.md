# REACT-ZERO: Technical Implementation Guide

**A multiplayer F-ZERO-inspired racing game built with React Three Fiber** requires mastering five interconnected domains: classic hover-racing mechanics, modern lobby systems, real-time networking, performant 3D rendering, and scalable React architecture. This guide synthesizes research across all five areas into actionable specifications for building REACT-ZERO.

## The energy-boost tradeoff defines F-ZERO's identity

F-ZERO's revolutionary mechanic uses a **single energy bar as both health AND boost fuel**, creating constant risk-reward tension. Players can boost anytime after lap 1, but each boost drains approximately **15-20% of their energy meter**. Running out of energy doesn't immediately kill you—but any collision at zero energy causes instant explosion. This creates psychological pressure where players must decide: "Do I boost now to overtake, risking death from the next wall tap?"

Energy recovery happens in **pit zones**—pink/magenta track strips typically near start/finish. Recovery is fast enough for full-speed drive-through refills. Crucially, boosting while in pits works because pit recovery rate exceeds boost drain rate at full health. Other hazards include:

- **Dash plates (zippers)**: Free speed boost, no energy cost—C-class boost regardless of ship stats
- **Land mines**: Significant damage + knockback; heavy ships can exploit "mine boosts" from collision physics
- **Rough terrain**: Slows vehicles significantly unless actively boosting through
- **Jump plates**: Launch vehicles airborne; hold down to nose-dive for faster return, hold up for distance
- **Guard rails**: Contact drains energy; advanced players "rail drift" for tighter corner lines

Ship differentiation uses four stats: **Body** (durability), **Boost** (duration/power), **Grip** (cornering), and **Weight** (affects collisions, top speed, acceleration tradeoff). Heavy ships knock lighter ships around and maintain momentum better; light ships accelerate faster and stay airborne longer.

## Hover physics without complex engines

F-ZERO's "floating" feel comes from treating vehicles as moving in **2D space relative to the track surface**, with height derived from the track mesh. Implementation approach:

```javascript
// Raycast-based hover simulation
const rayDown = new Vector3(0, -1, 0)
const targetHeight = 0.5 // hover height
const springConstant = 50
const damping = 5

useFrame((_, delta) => {
  raycaster.set(vehiclePos, rayDown)
  const hit = raycaster.intersectObject(trackMesh)[0]
  if (hit) {
    const heightError = targetHeight - hit.distance
    const springForce = heightError * springConstant
    const dampForce = -verticalVelocity * damping
    verticalVelocity += (springForce + dampForce) * delta
  }
  // Interpolate vehicle rotation to match track normal
  vehicleRef.current.up.lerp(hit.face.normal, 0.1)
})
```

What makes ships feel like hovering versus wheels: **lateral slide/drift** (vehicles slide more than wheeled vehicles), **gentle pitch/roll** with slight delay aligning to track surface, **momentum preservation** through direction changes, and **floaty steering response** with small delay between input and maximum turning rate.

## Lobby systems modeled on Mario Kart patterns

Mario Kart 8 Deluxe provides the gold standard for racing game lobbies. Key patterns to replicate:

**Room creation**: Hosts create private rooms and share **6-digit room codes** externally (Discord, text). Alternatively, friends can join directly from friend lists where hosts appear with checkered-flag icons. Room capacity maxes at **12 players** (humans + CPUs fill remaining slots). Host configures: speed class, items/powerups, race count (4-48), CPU difficulty (Easy/Normal/Hard).

**Ready-up flow**: Players select vehicle loadout (character → body → wheels → accessories), then press to confirm "Ready!" status. Host sees all ready states and initiates race start. Players can change loadout between races without leaving lobby.

**Track voting**: Each player sees **3 randomly presented tracks** plus a "Random" option. All votes submit simultaneously—then a **roulette animation** selects ONE random player's choice. If that player chose a track, everyone plays it; if they chose Random, the game picks from the full pool. This prevents strategic voting manipulation.

**Pre-race synchronization sequence**:
1. Track announced → Loading screen (wait for slowest player)
2. Track preview camera pan (skippable, but waits for all)
3. Starting grid placement based on standings or join order
4. Lakitu countdown: "3... 2... 1... GO!" (approximately 3-4 seconds)
5. **Rocket Start**: Press accelerator as "2" appears for speed boost; too early = stall

For mixed human/CPU races, CPUs use rubber-banding AI—speed adjusts based on player position. CPUs maintain "formations" that cap their speed and slow when approaching first place. Consider implementing rubber-banding only for AI behind players, not ahead.

## Server-authoritative multiplayer architecture

For 6 humans + 24 NPCs at 30+ FPS, use **WebSocket server-authoritative architecture with Colyseus.js**. This provides anti-cheat protection, simplified NPC synchronization, and acceptable latency for browser racing.

**Why not WebRTC peer-to-peer**: NAT traversal issues (symmetric NAT requires TURN relay anyway), cheat vulnerability without authoritative validation, mesh complexity (6 players = 15 connections each), and NPC synchronization becomes impossible without a central authority.

**Recommended tick rate**: **20-30Hz** provides smooth interpolation without overwhelming bandwidth. Valve's Source engine uses 20-30 updates/second for client sync while running simulation at 66 ticks internally.

**Entity state packet structure** (optimized binary, 12 bytes per entity):
| Field | Bytes | Notes |
|-------|-------|-------|
| Entity ID | 1 | Max 255 entities |
| Position X/Y/Z | 6 | Fixed-point, 0.1m precision |
| Rotation | 2 | Yaw only (0-360° mapped to uint16) |
| Velocity | 1 | Magnitude 0-255 |
| Steering | 1 | Angle int8 |
| Flags | 1 | Brake, boost, collision bits |

**Bandwidth calculation**: 30 entities × 12 bytes × 20Hz = **7.2 KB/s** upload from server. With delta compression (Colyseus native): ~3.5 KB/s per client. Server costs: Colyseus Cloud free tier for dev, ~$25-50/month production.

### Client-side prediction and interpolation

Local player prediction eliminates perceived input latency:

```javascript
// Send input with sequence number
const input = { seq: inputSeq++, steering, throttle, timestamp }
pendingInputs.push(input)
socket.send(input)
localCar.applyInput(input) // Immediate local prediction

// On server state arrival: reconcile
onServerState(state) {
  pendingInputs = pendingInputs.filter(i => i.seq > state.lastProcessedSeq)
  localCar.position = state.myPosition // Snap to server truth
  pendingInputs.forEach(input => localCar.applyInput(input)) // Replay unacked
}
```

Remote players render **100-150ms in the past** using interpolation buffers:

```javascript
const renderTime = Date.now() - INTERPOLATION_DELAY
const [prev, next] = findSurroundingStates(entity.history, renderTime)
const t = (renderTime - prev.time) / (next.time - prev.time)
entity.renderPosition = lerp(prev.position, next.position, t)
```

**Dead reckoning** for NPCs: `P₁ = P₀ + V₀·Δt + ½·A₀·Δt²`. Racing benefits from velocity-based prediction because vehicles follow predictable physics.

### Interest management for 30 entities

Not all clients need all 30 positions every tick:

| Priority | Update Rate | Entities |
|----------|-------------|----------|
| High | Every tick (20Hz) | Human players, nearby NPCs (<50m) |
| Medium | Every 2nd tick (10Hz) | Mid-range NPCs (50-150m) |
| Low | Every 4th tick (5Hz) | Distant NPCs (>150m) |

NPCs should be **server-simulated** (not deterministic on all clients) for guaranteed consistency—browser floating-point variance makes determinism unreliable across platforms.

## Three.js wireframe rendering and retro shaders

REACT-ZERO's visual identity relies on wireframe aesthetics with selective glow. Three.js wireframe options:

**EdgesGeometry** (recommended—shows hard edges only, no diagonals):
```jsx
function WireframeMesh({ geometry }) {
  const edges = useMemo(() => new THREE.EdgesGeometry(geometry, 15), [geometry])
  return (
    <lineSegments geometry={edges}>
      <lineBasicMaterial color="cyan" />
    </lineSegments>
  )
}
```

**Tron/vector aesthetic** via selective bloom—push color values above 1.0:
```jsx
// Glowing wireframe (renders bright for bloom pass)
<meshBasicMaterial color={[2, 4, 4]} toneMapped={false} />
```

**Post-processing pipeline** with @react-three/postprocessing:
```jsx
<EffectComposer>
  <Bloom 
    intensity={1.5}
    luminanceThreshold={0.9}  // Only bright objects glow
    mipmapBlur
  />
  <Noise opacity={0.02} />
  <Vignette offset={0.1} darkness={1.1} />
</EffectComposer>
```

**CRT/scanline shader** key components: screen curvature via UV distortion, scanlines via `sin(uv.y * resolution.y * PI)`, chromatic aberration by sampling RGB at offset UVs.

### Performance for 30+ entities

**Instanced meshes** are critical for NPCs:
```jsx
function NPCVehicles({ positions }) {
  const meshRef = useRef()
  const tempObj = useMemo(() => new THREE.Object3D(), [])
  
  useLayoutEffect(() => {
    positions.forEach((pos, i) => {
      tempObj.position.set(...pos)
      tempObj.updateMatrix()
      meshRef.current.setMatrixAt(i, tempObj.matrix)
    })
    meshRef.current.instanceMatrix.needsUpdate = true
  }, [positions])
  
  return (
    <instancedMesh ref={meshRef} args={[null, null, positions.length]}>
      <boxGeometry args={[1, 0.5, 2]} />
      <meshBasicMaterial wireframe color="magenta" />
    </instancedMesh>
  )
}
```

**Critical rule**: Never use `setState` in useFrame. Mutate refs directly:
```jsx
// ❌ Triggers React re-renders 60x/second
useFrame(() => setPosition(pos => pos + 0.1))

// ✅ Direct mutation via ref (no React overhead)
useFrame((_, delta) => meshRef.current.position.x += delta * speed)
```

### Track generation with splines

```javascript
const trackPoints = [
  new Vector3(0, 0, 0),
  new Vector3(50, 0, 30),
  new Vector3(100, 5, 0),
  new Vector3(50, 0, -30),
]
const curve = new CatmullRomCurve3(trackPoints, true) // closed loop
```

Track-relative collision uses `curve.getPointAt()` and `curve.getTangentAt()` to project vehicle position onto centerline and check lateral distance.

### Camera systems

Chase camera with smoothing and look-ahead:
```jsx
function ChaseCamera({ target, offset = [0, 5, -10] }) {
  useFrame((state, delta) => {
    // Calculate ideal position behind vehicle
    idealPos.copy(target.current.position)
      .add(new Vector3(...offset).applyQuaternion(target.current.quaternion))
    
    // Smooth follow with damping
    state.camera.position.lerp(idealPos, 1 - Math.pow(0.001, delta))
    
    // Look ahead based on velocity
    idealLookAt.copy(target.current.position)
      .add(target.current.velocity?.clone().multiplyScalar(0.5))
    state.camera.lookAt(idealLookAt)
  })
}
```

## React architecture and code organization

### State management with Zustand

Zustand dominates React game development because it works **outside React** (no Provider required):

```typescript
export const useGameStore = create<GameStore>((set, get) => ({
  playerPosition: [0, 0, 0],
  score: 0,
  gamePhase: 'menu',
  
  setPlayerPosition: (pos) => set({ playerPosition: pos }),
  incrementScore: (amount) => set((s) => ({ score: s.score + amount })),
}))

// In useFrame - read WITHOUT reactive binding
useFrame(() => {
  const pos = useGameStore.getState().playerPosition // No re-render trigger
  meshRef.current.position.set(...pos)
})
```

For high-frequency data (position, rotation, velocity): use **refs**. For UI-affecting state (score, lap count, game phase): use **Zustand state** but throttle updates to ~10fps.

### Recommended folder structure

```
/src
├── /assets               # Models, textures, shaders
├── /components
│   ├── /ui               # DOM overlays (HUD, Menu, Leaderboard)
│   └── /three            # Reusable R3F components
├── /features
│   ├── /vehicle          # Vehicle.tsx, useVehiclePhysics.ts, useControls.ts
│   ├── /race             # RaceManager.tsx, Checkpoint.tsx, useRaceState.ts
│   ├── /track            # TrackGenerator.tsx, TrackMesh.tsx
│   └── /multiplayer      # NetworkManager.tsx, useMultiplayer.ts, schemas/
├── /systems              # ECS-style systems
│   ├── PhysicsSystem.tsx
│   ├── CollisionSystem.tsx
│   └── NetworkSyncSystem.tsx
├── /stores               # Zustand stores
│   ├── gameStore.ts
│   └── networkStore.ts
├── /hooks                # Shared hooks
├── /utils                # Helpers, constants
├── /types                # TypeScript interfaces
├── App.tsx
├── Game.tsx              # Main game scene
└── main.tsx
```

**File length targets**: 100-200 lines per file, maximum 300 before refactoring. Extract logic into custom hooks (`useVehiclePhysics`, `useGameInput`). Separate types into `.types.ts` files.

### Component separation pattern

```tsx
function App() {
  return (
    <div className="game-container">
      {/* 3D Scene */}
      <Canvas>
        <Suspense fallback={<LoadingScreen />}>
          <Track />
          <Vehicle />
          <NPCVehicles />
          <ChaseCamera />
        </Suspense>
        <EffectComposer>{/* Post-processing */}</EffectComposer>
      </Canvas>
      
      {/* DOM Overlays - positioned absolutely over canvas */}
      <div className="ui-overlay">
        <HUD />
        <Minimap />
      </div>
    </div>
  )
}
```

## Implementation roadmap summary

**Phase 1: Core Racing**
- Implement hover physics with raycast + spring system
- Create spline-based track with CatmullRomCurve3
- Build energy/boost system with pit zones and dash plates
- Add wireframe rendering with EdgesGeometry + bloom

**Phase 2: Single-Player Polish**
- Ship differentiation (Body/Boost/Grip/Weight stats)
- AI opponents with rubber-banding
- Complete track hazards (mines, rough terrain, jump plates)
- Camera system with chase cam and shake effects

**Phase 3: Multiplayer**
- Colyseus server with race rooms at 20Hz tick rate
- Client-side prediction for local player
- Entity interpolation for remote players (100ms buffer)
- Lobby system with room codes and ready-up

**Phase 4: UX Polish**
- Track voting with roulette animation
- CRT/scanline post-processing shaders
- Countdown synchronization and rocket start
- Leaderboard and race results

**Key libraries**: @react-three/fiber, @react-three/drei, @react-three/postprocessing, zustand, colyseus.js (client), colyseus (server). Reference implementation: [pmndrs/racing-game](https://github.com/pmndrs/racing-game).