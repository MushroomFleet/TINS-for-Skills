# MechWarrior 2: Mercenaries — Technical Implementation Guide

**The 1996 mech simulator's strategic depth comes from five interconnected systems: heat management, location-based damage, independent torso control, weapon grouping, and tonnage-speed tradeoffs.** For an MVP capturing authentic "MechWarrior feel," heat management and location damage are non-negotiable—they transform combat from twitch shooting into tactical resource management. This guide provides exact formulas, numerical values, and behavioral specifications for React/Three.js implementation.

---

## Core formulas every system depends on

The simulation's mathematical foundation determines how mechs move, fight, and survive. These formulas should be implemented first as they cascade into nearly every other system.

**Speed calculation** (the most important formula):
```javascript
const maxSpeedKph = (engineRating / mechTonnage) * 16.2;
const walkSpeedKph = maxSpeedKph / 1.5;
const reverseSpeedKph = walkSpeedKph * 0.5;
```

**Heat balance per game tick:**
```javascript
const heatGenerated = weaponHeat + (throttlePercent * 0.02) + (jumpJetsActive ? jjCount * 2 : 0);
const heatDissipated = (singleHeatSinks * 1) + (doubleHeatSinks * 2) + environmentMod;
const netHeat = currentHeat + heatGenerated - heatDissipated;
```

**Armor points from tonnage:**
```javascript
const armorPoints = armorTonnage * 16; // Standard armor
const maxArmorPerLocation = internalStructure * 2; // Head exception: always 9 max
```

| Mech Tonnage | Internal Structure Weight | Max Total Armor | Typical Speed Range |
|--------------|---------------------------|-----------------|---------------------|
| 20-35 tons (Light) | 2-3.5 tons | 69-119 points | 97-130 kph |
| 40-55 tons (Medium) | 4-5.5 tons | 135-169 points | 64-97 kph |
| 60-75 tons (Heavy) | 6-7.5 tons | 195-231 points | 54-81 kph |
| 80-100 tons (Assault) | 8-10 tons | 259-307 points | 32-64 kph |

---

## Heat management creates tactical depth

Heat is what separates MechWarrior from standard shooters. Every offensive action generates heat; exceed capacity and your mech shuts down mid-combat. This forces players into constant risk/reward calculations.

**Heat thresholds and consequences:**

| Heat Level | Visual State | Mechanical Effect |
|------------|--------------|-------------------|
| 0-79% | Green gauge | Normal operation |
| 80-99% | Yellow gauge | Warning audio, approaching shutdown |
| 100% | Red gauge | **Automatic shutdown**—mech immobile, weapons offline |
| 100%+ (override) | Flashing red | Internal damage every ~1.5 seconds, ammo explosion risk |

**Base heat generation by weapon type:**

| Weapon | Heat | Damage | Range (m) | Recycle Time |
|--------|------|--------|-----------|--------------|
| Small Laser | 1 | 3 | 90 | ~4 seconds |
| Medium Laser | 3 | 5 | 270-350 | ~4 seconds |
| Large Laser | 8 | 8 | 450-850 | ~5 seconds |
| PPC | 10 | 10 | 540-570 | ~6 seconds |
| ER PPC (Clan) | 15 | 15 | 746-1000 | ~6 seconds |
| AC/20 | 7 | 20 | 270-400 | Fast |
| Gauss Rifle | 1 | 15 | 660-1820 | ~5 seconds |
| LRM-20 | 6 | 20 (1/missile) | 630-1000 | ~5 seconds |
| SRM-6 | 4 | 12 (2/missile) | 270-497 | ~3 seconds |
| Machine Gun | 0 | 2 | 90-175 | Continuous |

**Implementation specifics:**
- **10 base heat sinks** integrated in every mech (free in engine)
- Single heat sink: dissipates **1 heat/tick**, weighs 1 ton, 1 critical slot
- Double heat sink: dissipates **2 heat/tick**, weighs 1 ton, 3 slots (IS) / 2 slots (Clan)
- Movement generates heat: approximately `throttlePercent * 0.02` per tick
- Jump jets generate **~2 heat per jet per second** of flight
- **Environmental modifiers**: Arctic maps +20-50% cooling; Desert -20-30% cooling; Water submersion +75% for submerged heat sinks

---

## Location-based damage model

The eight-zone damage system creates emergent tactical play—players target specific components to cripple enemies strategically rather than simply depleting a health bar.

**Damage locations and kill conditions:**

| Location | Kill Condition | Destruction Effect |
|----------|----------------|-------------------|
| Head | Instant mech kill | Cockpit destroyed = pilot death |
| Center Torso | Instant mech kill | Engine destruction |
| Side Torso | Non-fatal | Attached arm destroyed; XL engine mechs die |
| Arms | Non-fatal | Arm weapons and equipment lost |
| Legs | Immobilizes | Mech cannot walk (can still twist, fire, use jump jets) |

**Internal structure points by tonnage** (critical for implementation):

| Tonnage | Head | Center Torso | Side Torso | Arm | Leg |
|---------|------|--------------|------------|-----|-----|
| 35 tons | 3 | 11 | 8 | 6 | 8 |
| 50 tons | 3 | 16 | 12 | 8 | 12 |
| 75 tons | 3 | 23 | 16 | 12 | 16 |
| 100 tons | 3 | 31 | 21 | 17 | 21 |

**Damage flow:**
1. Incoming damage hits **armor** first (1 damage = 1 armor point)
2. When armor depleted, damage transfers to **internal structure**
3. Internal structure damage triggers **critical hit rolls** on components
4. When structure reaches zero, location **destroyed**

**Critical hit effects:**
- **Engine hit**: +5 heat generation per hit (3 hits = engine destroyed)
- **Gyro hit**: Aim/stability penalties (2 hits = mech disabled)
- **Ammo hit**: 10% explosion chance, damage = remaining ammo value
- **Weapon hit**: Weapon destroyed immediately
- **Actuator hit**: Movement or aiming penalties per type

**HUD damage display (paperdoll):**
- Green/Blue → Yellow → Orange → Red → Black progression
- Each body section colored independently
- Alternative HTAL bar display shows exact armor/structure percentages

---

## Mech locomotion and torso independence

The independent torso twist—legs walking one direction while torso aims another—is the signature MechWarrior control feel. Heavy mechs should feel ponderous; lights should feel agile.

**Throttle system:**
- **10 discrete throttle settings** (keys 1-0 map to 0%-100%)
- Acceleration/deceleration scales inversely with tonnage
- Suggested formula: `acceleration = baseAccel * (100 / tonnage)`
- **Reverse speed**: ~50% of forward walking speed
- Turning radius increases with speed—must throttle down for tight turns

**Torso twist mechanics:**
- Maximum twist angle: **±90 degrees** from center
- Twist speed scales with engine rating (larger engine = faster twist)
- Torso twist allows engaging flanking enemies while maintaining course
- Keys: typically `<` and `>` for left/right twist
- **"M" key** aligns legs to torso facing

**Jump jets:**
- Each jet adds **30 meters** maximum jump distance
- Weight: 0.5 tons (light mechs), 1 ton (60-85 tons), 2 tons (assault)
- Maximum jets = walking movement points
- Heat generation: **1 heat per ~6 meters traveled**, minimum 3-5 heat per jump
- Fuel regenerates when not jumping (display as JTI gauge)
- Can fire weapons while airborne
- Death From Above (DFA): Landing on enemy head deals massive damage

**Example mech speeds:**

| Mech | Tonnage | Engine | Max Speed |
|------|---------|--------|-----------|
| Jenner | 35 | 245 | 113.4 kph |
| Centurion | 50 | 200 | 64.8 kph |
| Warhammer | 70 | 280 | 64.8 kph |
| Atlas | 100 | 300 | 48.6 kph |

---

## Weapon systems and grouping

Weapon grouping lets players fire coordinated salvos while managing heat. The system supports both "alpha strikes" (all weapons simultaneously) and chain-fire (sequential shots for sustained pressure).

**Weapon group system:**
- **5 weapon groups** assignable (SHIFT+1 through SHIFT+5)
- Toggle chain-fire vs group-fire with backslash (`\`)
- Group-fire: All weapons in group fire simultaneously (high burst, high heat)
- Chain-fire: Weapons fire sequentially (sustained damage, manageable heat)
- Weapon ready state shown by color: **Green** = ready, **Red** = cycling

**Missile lock mechanics:**
- LRMs require **lock-on** at ranges >75 meters (hold reticle on target ~2 seconds)
- Below 75m: LRMs fire dumb (no guidance)
- SRMs: No lock required, fire on iron sights
- Streak SRMs: Auto-lock at any range, no minimum
- Lock indicator: Reticle turns **red** when locked
- Lock can be maintained while twisting torso away (indirect fire over obstacles)

**Ammo specifications:**

| Ammo Type | Shots per Ton | Explosion Risk |
|-----------|---------------|----------------|
| AC/2 | 45 rounds | Yes |
| AC/5 | 20 rounds | Yes |
| AC/10 | 10 rounds | Yes |
| AC/20 | 5 rounds | Yes |
| Gauss | 8 rounds | No (but rifle explodes on crit) |
| LRM | 120 missiles | Yes |
| SRM | 100 missiles | Yes |
| Machine Gun | 200 rounds | Yes |

**CASE equipment** (0.5 tons, 1 slot): Contains ammo explosions to originating location only—critical for builds with volatile ammo.

---

## HUD elements for implementation

The interface communicates mech status through layered displays. Prioritize the paperdoll, heat gauge, and radar for MVP.

**Essential HUD components:**

1. **Damage paperdoll** (lower right): Wireframe mech showing color-coded damage per location
2. **Heat gauge** (vertical bar): Green→Yellow→Red progression with shutdown warning
3. **Radar** (circular): 250m/500m/1km/2km ranges, red enemies, green allies
4. **Weapon groups**: Current selection, ammo counts, cycling status
5. **Throttle indicator**: Current speed in kph, throttle percentage
6. **Targeting reticle**: Central crosshair, turns red on lock/in-range
7. **Target info**: Enemy mech wireframe showing their damage state

**"Bitchin' Betty" audio warnings:**
- "Heat level critical"
- "Warning—shutdown imminent"
- "Reactor meltdown imminent"
- "Critical hit!"
- "Weapon destroyed!"
- "Enemy power-up detected"

**Radar range cycle** (X key): 250m → 500m → 1000m → 2000m

---

## Combat feel: what makes MW2 tactical

MW2's **2-5 minute engagement times** and resource constraints create chess-like combat where positioning and timing matter more than reflexes.

**Pacing characteristics:**
- Armor must be worn down section by section—no one-shot kills on healthy mechs
- Alpha strikes (all weapons) cause instant overheating, forcing cooldown windows
- Successful players fire in controlled bursts, reposition during cooling
- "Circle of death" maneuver: Circle-strafe while trading fire, targeting weak sections

**Typical engagement ranges:**
- Most combat occurs at **150-300 meters** despite longer weapon ranges
- LRMs optimal at ~300-600m (lock time + travel time)
- PPCs effective to 1000m but slow projectiles easy to dodge
- Gauss rifles dominate at 600m+ (fast projectile, low heat)
- Brawling weapons (SRMs, AC/20, machine guns) under 150m

**Strategic targeting priorities:**
- **Legs**: "Legging" immobilizes target—extremely effective
- **Arms**: Removing arms strips 40-60% of most mechs' firepower
- **Rear torso**: Less armor than front, faster kills
- **Cockpit**: High-skill headshots for instant kills (certain mechs have easy profiles)

**The "walking tank" feel:**
- Heavy footstep rhythm (STOMP-STOMP-STOMP)
- Cockpit view bounces with each step
- Inertia prevents instant direction changes
- Independent torso creates smooth "turret on a chassis" sensation
- Screen static on taking PPC hits, visual distortion when overheating

---

## MVP prioritization: the essential five systems

For capturing authentic MechWarrior feel with minimal implementation, these systems are **non-negotiable**:

### 1. Heat management (CRITICAL)
Without heat, weapons have no cost and combat becomes standard shooter. Implement:
- Heat gauge filling on weapon fire
- Automatic shutdown at 100%
- Override option with damage penalty
- Basic dissipation over time

**Simplified MVP version**: Single heat bar, fixed dissipation rate, instant shutdown at max

### 2. Location-based damage with paperdoll (CRITICAL)
The 8-zone system creates tactical targeting. Implement:
- Separate armor/structure for each zone
- Visual paperdoll showing damage state
- Component destruction effects (arm lost = arm weapons gone)
- Instant kill on head/center torso destruction

**Simplified MVP version**: 5 zones (head, torso, left side, right side, legs), skip internal structure/crits

### 3. Independent torso twist (ESSENTIAL)
This is the signature control feel. Implement:
- Legs facing controlled by movement keys
- Torso facing controlled by mouse/aim direction
- ±90 degree twist limit
- Weapons fire from torso facing, not leg facing

### 4. Throttle-based movement with inertia (ESSENTIAL)
Mechs must feel heavy. Implement:
- Discrete throttle settings (not WASD instant movement)
- Acceleration/deceleration curves based on tonnage
- Slower turn rates at higher speeds
- Speed affects heat dissipation

### 5. Weapon grouping with cooldowns (IMPORTANT)
Fire control creates tactical variety. Implement:
- 3-5 weapon groups
- Per-weapon cooldown timers
- Visual feedback for weapon ready state
- Chain-fire vs group-fire toggle

---

## Reference mech configurations for testing

**Light Mech — Jenner (35 tons)**
- Engine 245 → 113.4 kph
- Weapons: 4× Medium Laser (12 heat total), 1× SRM-4 (3 heat)
- Heat sinks: 10
- Armor: ~100 points distributed
- Role: Fast striker, hit-and-run

**Medium Mech — Hunchback (50 tons)**
- Engine 200 → 64.8 kph
- Weapons: 1× AC/20 (7 heat, 20 damage), 2× Medium Laser (6 heat)
- Heat sinks: 13
- Armor: ~150 points distributed
- Role: Close-range brawler

**Heavy Mech — Warhammer (70 tons)**
- Engine 280 → 64.8 kph
- Weapons: 2× PPC (20 heat total, 20 damage), 2× Medium Laser
- Heat sinks: 18+
- Armor: ~200 points distributed
- Role: Sustained fire support

**Assault Mech — Atlas (100 tons)**
- Engine 300 → 48.6 kph
- Weapons: 1× AC/20, 1× LRM-20, 1× SRM-6, 4× Medium Laser
- Heat sinks: 20
- Armor: ~300 points (19 tons maximum)
- Role: Spearhead assault, command unit

---

## Implementation architecture notes for React/Three.js

**Suggested component separation:**

```
<MechSimulation>
  <ThrottleController /> // Handles speed, acceleration, inertia
  <TorsoController />    // Independent torso rotation
  <HeatManager />        // Heat generation, dissipation, shutdown
  <DamageModel />        // 8-zone armor/structure tracking
  <WeaponSystem />       // Groups, cooldowns, firing logic
  <HUDOverlay>           // 2D SVG overlay
    <Paperdoll />
    <HeatGauge />
    <RadarDisplay />
    <WeaponGroupsDisplay />
    <TargetingReticle />
  </HUDOverlay>
</MechSimulation>
```

**State management priorities:**
- Mech state (heat, damage per zone, throttle, torso angle) in centralized store
- Weapon states (cooldowns, ammo) as derived/computed values
- HUD components subscribe to relevant state slices
- Physics updates at fixed timestep (e.g., 60Hz), rendering decoupled

**Isometric camera considerations:**
- Torso twist should rotate mech model upper body only
- Lead indicators for projectile weapons at isometric angle
- Paperdoll always shows same orientation regardless of camera
- Consider "billboard" damage numbers floating in 3D space

This specification provides the numerical foundation for an authentic MW2: Mercenaries experience. Start with heat + damage + torso twist—these three systems alone will differentiate your game from standard shooters and capture the strategic mech combat feel that defined the franchise.