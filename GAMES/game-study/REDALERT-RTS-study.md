# Red Alert game mechanics: A technical blueprint for React RTS development

Red Alert's enduring design offers a masterclass in real-time strategy mechanics that can be directly translated to modern React/SVG implementations. The game's core systems—**cell-based terrain**, **sidebar construction**, **asymmetric faction balance**, and **fog of war**—create emergent strategic depth through relatively simple, well-defined rules. This report provides the specific values, formulas, and architectural patterns needed to implement an authentic Red Alert-inspired RTS in JavaScript.

## Resource gathering creates the economic foundation

The dual-resource system of **ore** (common, regenerating) and **gems** (rare, 2x value, non-regenerating) drives expansion and map control. Harvesters operate autonomously: collecting resources until full, returning to the nearest refinery, depositing credits, then repeating. This simple loop creates natural economic pressure without micromanagement burden.

**Key implementation values for harvesting:**
- Ore Truck capacity: **$700** (ore) / **$1,400** (gems)
- Ore Refinery cost: **$2,000**, provides **2,000 credit storage**, includes one free harvester
- Ore Silo cost: **$150**, adds **1,500 credit storage**
- Ore regeneration: Spreads from ore drills to adjacent tiles when density reaches maximum
- Harvester speed: 6 (relative units), HP: 600, self-repairs to 50%

The refinery converts raw resources to credits at a **1:1 ratio** for ore and **2:1 ratio** for gems. Storage overflow causes resource loss—if total credits exceed refinery plus silo capacity, excess is discarded. Destroying silos deducts stored credits immediately.

**Build time formula:** `Minutes = (Cost × 0.8) / 1000`. A $1,000 unit takes approximately 48 seconds. Multiple production buildings of the same type reduce build time proportionally, capped at 50% reduction with 2+ factories.

## Base building operates on a cell-based grid system

The Construction Yard serves as the central hub, enabling sidebar production within a **16-cell radius**. Buildings must be placed within **2 cells** of existing structures (walls extend this to 7 cells), creating organic base expansion that can chain across the map. Naval structures require placement entirely in water but can extend up to 8 cells from land buildings.

**Building footprint sizes vary systematically:**
| Structure Type | Footprint | Cost Range |
|----------------|-----------|------------|
| Power Plants | 2×2 to 2×3 | $300-$1,000 |
| Production Buildings | 3×3 | $500-$2,000 |
| Construction Yard | 4×4 | $2,500-$3,000 (MCV) |
| Walls/Turrets | 1×1 | $50-$600 |

The **power system** creates resource management tension. Each building consumes power (War Factory: -30, Tech Center: -200), while power plants generate it (Basic: +100, Advanced: +200). When consumption exceeds production, **production speed halves**, radar goes offline, and powered defenses (Tesla Coils, AA Guns) shut down. Damaged power plants produce proportionally less power based on remaining health percentage.

## Tech tree progression gates access to advanced capabilities

Prerequisites form dependency chains that pace technology access. The fundamental pattern: Construction Yard → Power → Barracks/Refinery → War Factory → Radar → Tech Center → Superweapons. Destroying prerequisite buildings prevents new production but doesn't affect existing units.

**Allied tech progression:**
```
Construction Yard
├── Power Plant ($300) → Advanced Power ($500)
├── Barracks ($300) → Pillbox, Turret
├── Ore Refinery ($2,000) → Radar Dome ($1,000)
├── War Factory ($2,000) → Service Depot ($1,200)
└── Tech Center ($1,500) → Chronosphere, Gap Generator, GPS Satellite
```

**Soviet tech progression:**
```
Construction Yard
├── Power Plant ($300) → Advanced Power ($500)
├── Barracks ($300) → Kennel, Flame Tower
├── Ore Refinery ($2,000) → Radar Dome ($1,000)
├── War Factory ($2,000) → Service Depot ($1,200)
└── Tech Center ($1,500) → Tesla Coil, Iron Curtain, Mammoth Tank
```

The Tech Center unlocks faction-defining late-game options: Allies gain the **Chronosphere** (teleportation), **GPS Satellite** (full map reveal), and **Gap Generator** (radar denial); Soviets unlock **Tesla Coils** (devastating defense), **Iron Curtain** (invulnerability), and the **Mammoth Tank** (self-healing super-heavy armor).

## Unit classification follows clear combat roles

Units divide into four categories—infantry, vehicles, aircraft, and naval—each with distinct movement types and combat characteristics. The **SpeedType** system determines terrain passability: Foot (infantry), Track (tanks, crushes infantry), Wheel (light vehicles), Float (naval), Winged (ignores terrain).

**Infantry unit archetypes and stats:**
| Unit | Cost | HP | Speed | Role |
|------|------|-----|-------|------|
| Rifleman | $100 | 50 | 4 | Basic combat |
| Rocket Soldier | $300 | 45 | 3 | Anti-vehicle/air |
| Engineer | $500 | 25 | 4 | Building capture |
| Tanya | $1,200 | 100 | 5 | Hero unit, C4 demolition |

**Vehicle unit archetypes and stats:**
| Unit | Cost | HP | Speed | Armor | Role |
|------|------|-----|-------|-------|------|
| Light Tank | $700 | 300 | 9 | Heavy | Fast assault |
| Medium Tank | $800 | 400 | 8 | Heavy | Main battle tank |
| Heavy Tank (Soviet) | $950 | 400 | 7 | Heavy | Superior firepower |
| Mammoth Tank | $1,700 | 600 | 4 | Heavy | Super-heavy, self-heals to 50% |
| Artillery | $600 | 75 | 6 | Light | Long-range siege |
| Harvester | $1,400 | 600 | 6 | Heavy | Resource collection |

Aircraft operate from dedicated structures (Helipads, Airfields) and must return to rearm. Naval units divide between surface combat (Destroyers, Cruisers) and stealth submarine warfare, with explicit counter relationships: Destroyers counter submarines via depth charges, submarines counter surface ships via torpedoes.

## Combat mechanics use a warhead-versus-armor damage matrix

Damage calculation follows a deterministic formula: **Final Damage = Base Damage × Warhead Verses% × Prone Modifier**. The system uses **11 armor types** (none, flak, plate, light, medium, heavy, wood, steel, concrete, special_1, special_2) with each warhead defining effectiveness percentages against each type.

**Example warhead configurations:**
- **HollowPoint** (Sniper/Tanya): 200% vs unarmored infantry, 1% vs vehicles/buildings
- **AP (Armor Piercing)**: 50% vs infantry, 100% vs vehicles, 75% vs buildings
- **0% Verses = Cannot target (even with force-fire)**
- **1% Verses = Won't auto-acquire but can be force-fired**

Area-of-effect weapons use **CellSpread** for radius and **PercentAtMax** for edge damage falloff. A nuclear missile with CellSpread=10 and PercentAtMax=0.5 deals full damage at impact, falling linearly to 50% at 10 cells away.

**Targeting priority algorithm:**
1. Threatening units (currently attacking this unit)
2. Armor type effectiveness (higher Verses% = higher priority)
3. Distance (closer targets preferred)
4. Target value (production buildings > defenses > other)

Units won't auto-acquire targets with ≤1% Verses value. Prone infantry receive only **50% damage** (configurable). Rate of fire is measured in game frames (15 frames = 1 second at normal speed).

## Faction asymmetry creates strategic diversity through complementary strengths

Red Alert's defining design achievement is **asymmetric balance**—factions feel completely different while remaining competitively viable. The core trade-off: Allies favor **speed and versatility**, Soviets favor **raw power and durability**.

**Allied faction identity:**
- Faster, cheaper units (Medium Tank: $800, 400 HP, speed 8)
- **Naval dominance** with Cruisers ($2,000) and Destroyers ($1,000)
- Information warfare: GPS Satellite reveals map, Gap Generator denies enemy radar
- Special operations: Spy infiltration, Thief credit theft, Tanya demolition
- Chronosphere enables strategic teleportation

**Soviet faction identity:**
- Slower, more powerful units (Heavy Tank: $950, 400 HP, twin cannons)
- **Ground superiority** with Mammoth Tank ($1,700, 600 HP, self-healing)
- **Air force** (MiGs, Yaks, Hinds) versus Allied helicopter-only aviation
- Tesla technology: Tesla Coils one-shot most tanks, Shock Troopers can't be crushed
- Iron Curtain grants temporary invulnerability for breakthrough attacks

**Balance through domain specialization:**
| Domain | Allied Advantage | Soviet Advantage |
|--------|------------------|------------------|
| Ground | Speed, numbers | Raw firepower |
| Naval | **Dominant** | Submarines only |
| Air | Limited (RA1) | **Full air force** |
| Defense | Information denial | Tesla Coils |
| Early Game | Disadvantaged | Tank rush |
| Late Game | Naval/intel | Disadvantaged |

The counter-play design ensures no strategy is unbeatable: Tesla Coils counter tank rushes, but Allies can use air or special operations; Cruisers dominate seas, but Soviets attack from land/air; Spy infiltration is countered by Attack Dogs.

## Building placement rules constrain strategic expansion

The **Construction Yard** establishes a 16-cell build radius. The **2-cell adjacency rule** prevents isolated forward bases without significant infrastructure investment. Naval buildings can reach 8 cells from existing structures but must be entirely in water.

**Terrain types and their effects:**
| Terrain | Buildable | Foot Speed | Track Speed | Wheel Speed |
|---------|-----------|------------|-------------|-------------|
| Clear | Yes | 100% | 100% | 100% |
| Road | Yes | 100% | 100% | 100% |
| Rough | No | Reduced | Reduced | Reduced |
| Water | No | 0% | 0% | 0% |
| Ore | No | 100% | 100% | 100% |
| Cliff | No | 0% | 0% | 0% |

Bridges are destructible via force-attack (Ctrl+Click) and in RA2+ can be repaired by Engineers entering repair huts. All ground units can cross intact bridges; naval units pass underneath.

**Defense structure effectiveness:**
| Defense | Cost | Power | Damage Type | Notes |
|---------|------|-------|-------------|-------|
| Pillbox | $400 | -15 | Anti-infantry | Rapid-fire, can't fire over walls |
| Turret | $600 | -40 | Anti-vehicle | 105mm cannon |
| Tesla Coil | $1,500 | -150 | All-purpose | Instantly kills infantry, high anti-armor |
| AA Gun | $600 | -40 | Anti-air | Requires power |

Walls ($50/section) block pathing and tank shells. Sandbags and barbed wire are crushable by tracked vehicles; concrete walls are not.

## Superweapons provide game-changing strategic options

Both factions access devastating late-game abilities through Tech Center prerequisites and significant power investment.

**Chronosphere (Allied, $2,800, -200 power):**
- Teleports one vehicle anywhere on map (RA1) or 3×3 area (RA2)
- Units teleport back after set duration in RA1
- 20% chance of Chronal Vortex causing 33% area damage (RA1)
- **Kills infantry** caught in effect
- Can teleport enemy vehicles into water for instant destruction

**Iron Curtain (Soviet, $2,800, -200 power):**
- Grants invulnerability for ~50 seconds (RA2)
- Affects vehicles and structures only
- **Instantly kills infantry** if applied to them
- Removes Terror Drone infections
- Cannot affect air units or units in transports
- Demolition Trucks explode immediately when Iron Curtained

**Nuclear Missile (~$2,500, 10-minute charge):**
- CellSpread=10, massive area damage
- Creates lasting radiation field
- Primary target priority: War Factory (AI behavior)

## The sidebar interface organizes all player actions

The iconic right-side sidebar remains the genre standard for good reason: it provides persistent access to production, power monitoring, and tactical commands without obscuring the battlefield.

**Sidebar layout (top to bottom):**
1. **Credits display** - Current funds, updates in real-time
2. **Minimap** (128×128 pixels at 640×480) - Shows faction logo when radar unavailable
3. **Repair/Sell buttons** - Wrench and dollar icons
4. **Build tabs** (RA2+): Q=Structures, W=Defenses, E=Infantry, R=Vehicles
5. **Build grid** - 2-3 columns of 48×48 pixel buttons with progress overlays

**Build button states:**
- **Available**: Full color, clickable
- **Building**: Progress bar overlay filling from bottom
- **On Hold**: Paused indicator (right-click to pause/resume)
- **Ready**: Pulsing highlight, awaiting placement
- **Unavailable**: Grayscale with lock icon, shows prerequisites on hover

**Power bar**: Thin vertical meter on sidebar edge showing production (green) versus consumption (red). When red exceeds green, production slows 50% and powered structures fail.

## Minimap and fog of war reveal information asymmetrically

The minimap requires a **Radar Dome/Tower** to function—without it, players see only their faction logo. This creates a critical early-game decision: invest in radar for information advantage or rush military production.

**Fog of war implementation:**
- **Shroud** (unexplored): Completely black, terrain unknown—permanently removed once explored
- **Fog** (explored but not visible): Semi-transparent gray—terrain visible, buildings shown as "last seen," **enemy units hidden**
- Each unit has defined **sight range** (infantry ~4-6 cells, vehicles ~6-8 cells)
- Buildings provide persistent vision while standing
- GPS Satellite/Spy Satellite reveals entire map permanently

**Minimap displays:**
- Terrain topology and resource deposits
- Buildings (both friendly and enemy-last-seen)
- Units as colored dots matching player colors
- Camera viewport indicator (white rectangle)
- Left-click moves camera; some versions support right-click unit orders

## Unit selection supports both casual and advanced play

The selection system accommodates different skill levels through layered controls:

**Basic selection:**
- Left-click: Select single unit
- Shift+click: Add/remove from selection
- Click-drag: Box/marquee selection
- Double-click: Select all visible units of same type

**Control groups (0-9):**
- Ctrl+number: Assign selected units to group
- Press number: Select that group
- Double-tap number: Center camera on group

**Advanced shortcuts:**
- T: Select all same-type units on screen
- TT: Select all same-type units on map
- P: Select all units on screen
- N: Cycle to next unit

Maximum selection is effectively unlimited within the game's unit caps (~500 per category).

## Game flow differs between campaign and skirmish modes

**Campaign missions** feature 14 missions per faction with diverse objectives: destruction (eliminate all enemies), capture (take specific building), defense (protect target for duration), infiltration (spy entry), escort, and time-limited challenges. FMV briefings between missions advance the Cold War alternate-history narrative. Scripted events trigger reinforcements, enemy attacks, and cinematic moments.

**Skirmish mode** provides replayable competitive scenarios with configurable options:
- Starting credits: $5,000 / $10,000 / $20,000
- Superweapons: On/Off
- Crates (power-ups): On/Off
- Shroud: On/Off
- Short Game: Buildings only vs. full annihilation
- AI difficulty: Easy/Medium/Hard

**Win conditions:**
- **Standard**: Destroy all enemy units AND structures
- **Short Game**: Destroy all enemy buildings only
- **Campaign**: Complete mission-specific objectives

**Lose conditions:**
- All player structures destroyed
- All player units destroyed
- Mission-specific failure triggers

## Implementation architecture for React/SVG

The research suggests a component-based architecture mapping directly to Red Alert's systems:

**Core data structures:**
```javascript
// Cell-based map grid
const cell = { terrain: 'clear', ore: 0, shroud: true, visible: false };

// Building with prerequisites
const building = {
  id: 'warFactory',
  footprint: { width: 3, height: 3 },
  cost: 2000,
  power: -30,
  prerequisites: ['oreRefinery', 'barracks'],
  unlocks: ['mediumTank', 'harvester']
};

// Damage calculation
function calculateDamage(weapon, target) {
  const armorIndex = ARMOR_TYPES[target.armor];
  const verses = weapon.warhead.verses[armorIndex] / 100;
  let damage = weapon.baseDamage * verses;
  if (target.isProne) damage *= 0.5;
  return Math.floor(damage);
}
```

**Recommended React component hierarchy:**
- GameContainer (state management, game loop)
  - Battlefield (SVG/Canvas isometric view)
    - TerrainLayer, FogLayer, UnitLayer, BuildingLayer
  - Sidebar
    - CreditsDisplay, Minimap, PowerBar, BuildTabs, BuildGrid
  - SelectionPanel (selected unit info, commands)

**Key state management considerations:**
- Cell visibility updates each tick based on unit sight ranges
- Build progress advances based on time and power status
- Unit AI runs targeting priority algorithm for auto-attack
- Harvester pathfinding loops: find ore → collect → return → deposit → repeat

The 48-second-per-$1,000 build time formula, 16-cell build radius, and warhead-versus-armor damage matrix provide the mathematical foundation for authentic Red Alert gameplay in a modern web environment.