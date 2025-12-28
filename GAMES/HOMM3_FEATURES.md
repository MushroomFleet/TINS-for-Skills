# Heroes of Might and Magic III — Complete Game Features Specification

> **TINS-Compliant Documentation** | React JSX with SVG Graphics | Modular Component Architecture

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Core Game Mechanics](#core-game-mechanics)
3. [Map & Exploration System](#map--exploration-system)
4. [Town & Castle System](#town--castle-system)
5. [Hero System](#hero-system)
6. [Creature & Army System](#creature--army-system)
7. [Combat System](#combat-system)
8. [Magic System](#magic-system)
9. [Resource Economy](#resource-economy)
10. [Victory Conditions](#victory-conditions)
11. [Technical Architecture](#technical-architecture)
12. [Component Specifications](#component-specifications)
13. [SVG Graphics System](#svg-graphics-system)
14. [Data Models](#data-models)
15. [State Management](#state-management)
16. [Accessibility & Performance](#accessibility--performance)

---

## Project Overview

### Description

A browser-based implementation of the classic turn-based strategy game *Heroes of Might and Magic III*, featuring procedurally generated adventure maps, tactical hex-based combat, town building, hero development, and army management. Built with React JSX using SVG graphics for scalable, resolution-independent visuals and a modular component-based architecture.

### Key Features

- **Turn-Based Strategy**: Players take turns moving heroes, building structures, and managing resources
- **Adventure Map Exploration**: Navigate procedurally generated worlds with fog-of-war
- **Tactical Combat**: Hex-grid battlefield with unit positioning and spell casting
- **Town Development**: Build and upgrade structures to unlock units and abilities
- **Hero Progression**: Level up heroes, learn skills, collect artifacts
- **Army Composition**: Recruit and manage diverse creature types with unique abilities
- **Magic System**: Schools of magic with tiered spells and mana management
- **Multiplayer Support**: Hot-seat and async multiplayer modes

### Target Platform

Modern web browsers (Chrome, Firefox, Safari, Edge) with responsive design supporting desktop (1920×1080 primary) down to tablet (1024×768 minimum).

---

## Core Game Mechanics

### Turn Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    TURN PHASES                               │
├─────────────────────────────────────────────────────────────┤
│  1. UPKEEP PHASE                                            │
│     ├── Resource collection (mines, towns)                  │
│     ├── Creature growth in towns                            │
│     ├── Spell effects tick/expire                           │
│     └── Movement points refresh                             │
│                                                             │
│  2. ACTION PHASE                                            │
│     ├── Move heroes on adventure map                        │
│     ├── Interact with map objects                           │
│     ├── Build structures in towns                           │
│     ├── Recruit creatures                                   │
│     ├── Trade between heroes                                │
│     └── Cast adventure spells                               │
│                                                             │
│  3. COMBAT PHASE (triggered by encounters)                  │
│     ├── Tactical hex-grid battles                           │
│     ├── Spell casting                                       │
│     └── Victory/defeat resolution                           │
│                                                             │
│  4. END PHASE                                               │
│     ├── Check victory conditions                            │
│     └── Pass turn to next player                            │
└─────────────────────────────────────────────────────────────┘
```

### Movement System

| Terrain Type | Movement Cost | Description |
|--------------|---------------|-------------|
| Road | 0.5 | Fastest travel, connects towns |
| Grass | 1.0 | Standard terrain |
| Dirt | 1.0 | Standard terrain |
| Sand | 1.25 | Slightly slower |
| Snow | 1.5 | Difficult terrain |
| Swamp | 1.75 | Very difficult |
| Mountain | Impassable | Cannot cross |
| Water | Boat required | Naval movement |
| Underground | 1.0 | Subterranean caves |

**Movement Points**: Heroes have base movement determined by:
- Slowest creature in army
- Hero skills (Logistics, Pathfinding, Navigation)
- Artifacts equipped
- Terrain modifiers

```javascript
// Movement calculation formula
movementPoints = baseMovement × (1 + logisticsBonus) × terrainModifier × artifactBonus
```

---

## Map & Exploration System

### Adventure Map Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    MAP LAYERS                                │
├─────────────────────────────────────────────────────────────┤
│  LAYER 1: Terrain Grid                                      │
│           └── Tile-based, determines movement costs         │
│                                                             │
│  LAYER 2: Roads & Rivers                                    │
│           └── Overlay paths, water features                 │
│                                                             │
│  LAYER 3: Objects & Structures                              │
│           ├── Towns, Mines, Dwellings                       │
│           ├── Interactive objects                           │
│           └── Decorative elements                           │
│                                                             │
│  LAYER 4: Mobile Units                                      │
│           ├── Heroes                                        │
│           └── Wandering monsters                            │
│                                                             │
│  LAYER 5: Fog of War                                        │
│           ├── Unexplored (black)                            │
│           ├── Previously seen (grayed)                      │
│           └── Currently visible (clear)                     │
└─────────────────────────────────────────────────────────────┘
```

### Map Object Categories

#### Resource Structures (Capturable)

| Object | Resource | Daily Production | Defense |
|--------|----------|------------------|---------|
| Gold Mine | Gold | 1000/day | 500 gold worth |
| Ore Pit | Ore | 2/day | 300 gold worth |
| Sawmill | Wood | 2/day | 300 gold worth |
| Gem Pond | Gems | 1/day | 750 gold worth |
| Crystal Cavern | Crystal | 1/day | 750 gold worth |
| Sulfur Dune | Sulfur | 1/day | 750 gold worth |
| Alchemist Lab | Mercury | 1/day | 750 gold worth |

#### Interactive Objects

| Object | Effect | Revisitable |
|--------|--------|-------------|
| Treasure Chest | 1500 gold OR 1000 exp | No |
| Artifact | Equip to hero | No |
| Resource Pile | Gain resources | No |
| Learning Stone | +1000 exp | No |
| School of Magic | Learn spell | Yes (cost) |
| University | Learn skill | Yes (cost) |
| Trading Post | Exchange resources | Yes |
| Obelisk | Reveal puzzle map | No |
| Seer's Hut | Quest giver | Quest-based |
| Shrine | +1 to primary stat | No |
| Fountain of Youth | Refresh movement | Weekly |
| Stables | +400 movement | Weekly |
| Watering Hole | +400 movement | Weekly |

#### Creature Dwellings

External dwellings provide creature recruitment outside of towns:

| Dwelling | Creature | Growth Rate |
|----------|----------|-------------|
| Hovel | Peasant | 14/week |
| Archer's Tower | Archer | 9/week |
| Griffin Tower | Griffin | 7/week |
| Barracks | Swordsman | 6/week |
| Training Grounds | Cavalier | 4/week |
| Dragon Cave | Dragon | 1/week |

### Fog of War Mechanics

```javascript
// Visibility calculation
const visibilityRadius = hero.scoutingSkill > 0 
  ? baseRadius + (hero.scoutingSkill * 2) 
  : baseRadius;

// States
VISIBILITY_STATES = {
  UNEXPLORED: 0,      // Never seen - completely black
  SHROUDED: 1,        // Previously seen - dimmed, shows terrain/buildings
  VISIBLE: 2          // Currently in sight - full detail, shows units
}
```

---

## Town & Castle System

### Town Types (Factions)

Each faction has a unique town with thematic creatures and buildings:

| Faction | Theme | Specialty | Native Terrain |
|---------|-------|-----------|----------------|
| Castle | Human Knights | Balanced, strong morale | Grass |
| Rampart | Elven Nature | Magic resistance, healing | Forest |
| Tower | Wizards | Powerful magic, constructs | Snow |
| Inferno | Demons | Fire magic, summoning | Lava |
| Necropolis | Undead | Raise dead, no morale | Cursed |
| Dungeon | Dark Elves | Speed, assassination | Underground |
| Stronghold | Barbarians | High attack, rage | Wasteland |
| Fortress | Swamp Creatures | Poison, defense | Swamp |
| Conflux | Elementals | Elemental magic | Any |

### Building System

#### Common Buildings (All Factions)

```
┌─────────────────────────────────────────────────────────────┐
│                    BUILDING TREE                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  TOWN HALL ──► CITY HALL ──► CAPITOL                        │
│  (500g)        (2500g)       (10000g)                       │
│  +500/day      +1000/day     +4000/day                      │
│                                                             │
│  FORT ──► CITADEL ──► CASTLE                                │
│  (5000g)  (2500g)     (5000g)                               │
│  +50% def  +100% def  +200% def, 2 towers                   │
│                                                             │
│  TAVERN ──► BLACKSMITH ──► MARKETPLACE                      │
│  (500g)     (1000g)        (500g)                           │
│  Hire heroes Ballista      Trade resources                  │
│                                                             │
│  MAGE GUILD (Levels 1-5)                                    │
│  Level 1: 2000g, 5 wood, 5 ore                              │
│  Level 2: 1000g, 5 wood, 5 ore, 4 crystals                  │
│  Level 3: 1000g, 5 wood, 5 ore, 6 crystals, 2 gems          │
│  Level 4: 1000g, 5 wood, 5 ore, 8 crystals, 4 gems          │
│  Level 5: 1000g, 5 wood, 5 ore, 10 crystals, 6 gems         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Creature Dwellings

Each town has 7 tiers of creature dwellings:

```javascript
// Castle faction example
CASTLE_DWELLINGS = {
  tier1: { 
    name: "Guardhouse", 
    creature: "Pikeman", 
    upgradedCreature: "Halberdier",
    cost: { gold: 400, wood: 5 },
    growth: 14 
  },
  tier2: { 
    name: "Archer's Tower", 
    creature: "Archer", 
    upgradedCreature: "Marksman",
    cost: { gold: 1000, wood: 5, ore: 5 },
    growth: 9 
  },
  tier3: { 
    name: "Griffin Tower", 
    creature: "Griffin", 
    upgradedCreature: "Royal Griffin",
    cost: { gold: 1000, wood: 5 },
    growth: 7 
  },
  tier4: { 
    name: "Barracks", 
    creature: "Swordsman", 
    upgradedCreature: "Crusader",
    cost: { gold: 2000, ore: 5 },
    growth: 4 
  },
  tier5: { 
    name: "Monastery", 
    creature: "Monk", 
    upgradedCreature: "Zealot",
    cost: { gold: 2000, wood: 5, ore: 5 },
    growth: 3 
  },
  tier6: { 
    name: "Training Grounds", 
    creature: "Cavalier", 
    upgradedCreature: "Champion",
    cost: { gold: 3000, wood: 10, ore: 10 },
    growth: 2 
  },
  tier7: { 
    name: "Portal of Glory", 
    creature: "Angel", 
    upgradedCreature: "Archangel",
    cost: { gold: 20000, wood: 20, ore: 20, gems: 10, crystals: 10 },
    growth: 1 
  }
}
```

### Build Queue Rules

1. **One building per turn** per town
2. **Prerequisites must be met** (building tree dependencies)
3. **Resources deducted immediately** upon construction start
4. **Building completes at turn end**
5. **Capitol limit**: Only one Capitol across all towns

---

## Hero System

### Hero Classes

| Class | Primary Stat Focus | Special Ability |
|-------|-------------------|-----------------|
| Knight | Attack | +1 Leadership/level |
| Cleric | Power | +1 Wisdom/level |
| Ranger | Attack | +350 movement |
| Druid | Power | Nature spell specialization |
| Alchemist | Power | +350 movement |
| Wizard | Power | +1 Spell Power/level |
| Demoniac | Power | Fire magic mastery |
| Heretic | Power | +1 Mana/level |
| Death Knight | Attack | Necromancy bonus |
| Necromancer | Power | Necromancy specialization |
| Overlord | Attack | +1 Leadership/level |
| Warlock | Power | Magic resistance |
| Barbarian | Attack | +1 Attack/level |
| Battle Mage | Attack | Ballistics bonus |
| Beastmaster | Attack | Creature bonus |
| Witch | Power | Eagle Eye bonus |
| Planeswalker | Power | Elemental immunity |
| Elementalist | Power | Elemental magic |

### Primary Statistics

| Stat | Effect | Base Value |
|------|--------|------------|
| Attack | +5% damage per point | 0-99 |
| Defense | -5% damage received per point | 0-99 |
| Power | Spell damage/effect multiplier | 0-99 |
| Knowledge | Maximum mana = Knowledge × 10 | 0-99 |

### Secondary Skills (28 Total)

Skills have three mastery levels: **Basic**, **Advanced**, **Expert**

```
┌─────────────────────────────────────────────────────────────┐
│                    SKILL CATEGORIES                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  COMBAT SKILLS                                              │
│  ├── Archery (ranged damage +10/25/50%)                     │
│  ├── Armorer (damage reduction +5/10/15%)                   │
│  ├── Offense (melee damage +10/20/30%)                      │
│  ├── Tactics (deployment hex range +1/3/5)                  │
│  ├── Leadership (morale +1/2/3)                             │
│  ├── Luck (luck +1/2/3)                                     │
│  └── Resistance (spell resistance +5/10/20%)                │
│                                                             │
│  MAGIC SKILLS                                               │
│  ├── Wisdom (learn level 3/4/5 spells)                      │
│  ├── Intelligence (+25/50/100% mana)                        │
│  ├── Scholar (share spells with allied heroes)              │
│  ├── Eagle Eye (chance to learn enemy spells)               │
│  ├── Mysticism (+2/3/4 mana regen/day)                      │
│  ├── Sorcery (+5/10/15% spell damage)                       │
│  ├── Air Magic (enhanced air spells)                        │
│  ├── Earth Magic (enhanced earth spells)                    │
│  ├── Fire Magic (enhanced fire spells)                      │
│  └── Water Magic (enhanced water spells)                    │
│                                                             │
│  ADVENTURE SKILLS                                           │
│  ├── Logistics (+10/20/30% movement)                        │
│  ├── Pathfinding (terrain penalty reduced)                  │
│  ├── Navigation (+50/100/150% sea movement)                 │
│  ├── Scouting (+1/2/3 visibility radius)                    │
│  ├── Estates (+125/250/500 gold/day)                        │
│  ├── Diplomacy (surrender discount, join chance)            │
│  ├── Learning (+5/10/15% exp gain)                          │
│  ├── Necromancy (+5/10/15% skeletons raised)                │
│  ├── First Aid (+50/100/150 tent healing)                   │
│  ├── Artillery (ballista damage/accuracy)                   │
│  └── Ballistics (catapult effectiveness)                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Hero Experience Table

| Level | Experience Required | Skill Slots |
|-------|---------------------|-------------|
| 1 | 0 | 2 |
| 2 | 1,000 | 3 |
| 3 | 2,000 | 3 |
| 4 | 3,200 | 4 |
| 5 | 4,600 | 4 |
| 6 | 6,200 | 5 |
| 7 | 8,000 | 5 |
| 8 | 10,000 | 6 |
| 9 | 12,200 | 6 |
| 10 | 14,700 | 7 |
| ... | +15% per level | Max 8 |

### Artifacts System

#### Artifact Slots

```
┌─────────────────────────────────────────────────────────────┐
│                    HERO EQUIPMENT SLOTS                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│      ┌─────┐                     ┌─────┐                    │
│      │ HEAD│                     │NECK │                    │
│      └─────┘                     └─────┘                    │
│                                                             │
│  ┌─────┐  ┌─────────────┐  ┌─────┐                          │
│  │RIGHT│  │    TORSO    │  │LEFT │                          │
│  │HAND │  │             │  │HAND │                          │
│  └─────┘  └─────────────┘  └─────┘                          │
│                                                             │
│      ┌─────┐         ┌─────┐                                │
│      │RIGHT│         │LEFT │                                │
│      │RING │         │RING │                                │
│      └─────┘         └─────┘                                │
│                                                             │
│      ┌─────┐         ┌─────┐                                │
│      │FEET │         │MISC │ × 5                            │
│      └─────┘         └─────┘                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Artifact Tiers

| Tier | Rarity | Typical Bonus |
|------|--------|---------------|
| Treasure | Common | +1-2 stat |
| Minor | Uncommon | +2-3 stat |
| Major | Rare | +3-4 stat or ability |
| Relic | Epic | +4-6 stat or powerful ability |
| Combination | Legendary | Assembled from parts |

#### Example Artifacts

```javascript
ARTIFACTS = {
  // Treasure Tier
  centaurAxe: {
    slot: "weapon",
    tier: "treasure",
    effects: { attack: 2 }
  },
  
  // Minor Tier
  armorOfWonder: {
    slot: "torso",
    tier: "minor",
    effects: { attack: 1, defense: 1, power: 1, knowledge: 1 }
  },
  
  // Major Tier
  capeOfConjuring: {
    slot: "shoulders",
    tier: "major",
    effects: { spellDuration: 3 }
  },
  
  // Relic Tier
  armageddonsBlade: {
    slot: "weapon",
    tier: "relic",
    effects: { 
      attack: 3, 
      defense: 3, 
      power: 3, 
      knowledge: 6,
      spells: ["armageddon"],
      immunity: ["armageddon"]
    }
  },
  
  // Combination (Angelic Alliance)
  angelicAlliance: {
    components: ["armorOfWonder", "sandalsOfSaint", "celestialNecklace", 
                 "lionShield", "swordOfJudgement", "helmOfHeavenlyEnlightenment"],
    slot: "combination",
    tier: "combination",
    effects: {
      attack: 21, defense: 21, power: 21, knowledge: 21,
      allSpells: true,
      airImmunity: true, earthImmunity: true, 
      fireImmunity: true, waterImmunity: true
    }
  }
}
```

---

## Creature & Army System

### Creature Statistics

Each creature has the following stats:

| Stat | Description |
|------|-------------|
| Attack | Offensive power (damage calculation) |
| Defense | Defensive power (damage reduction) |
| Damage | Min-Max damage range |
| Health | Hit points per creature |
| Speed | Initiative and movement on hex grid |
| Growth | Weekly recruitment availability |
| Cost | Gold and resources to recruit |
| Abilities | Special combat abilities |

### Example Creature Data

```javascript
// Castle Tier 7 - Angel/Archangel
CREATURES = {
  angel: {
    tier: 7,
    attack: 20,
    defense: 20,
    damage: { min: 50, max: 50 },
    health: 200,
    speed: 12,
    growth: 1,
    cost: { gold: 3000, gems: 1 },
    abilities: ["flying", "hateDevils"],
    size: 2, // hexes
    ranged: false,
    upgraded: false
  },
  
  archangel: {
    tier: 7,
    attack: 30,
    defense: 30,
    damage: { min: 50, max: 50 },
    health: 250,
    speed: 18,
    growth: 1,
    cost: { gold: 5000, gems: 3 },
    abilities: ["flying", "hateDevils", "resurrection"],
    size: 2,
    ranged: false,
    upgraded: true
  }
}
```

### Creature Abilities

| Ability | Effect |
|---------|--------|
| Flying | Can move over obstacles, no retaliation from ground |
| Teleporting | Can appear anywhere on battlefield |
| Two-Hex Attack | Attacks two adjacent hexes |
| Breath Attack | Attacks in a line |
| No Retaliation | Enemy can't counter-attack |
| Unlimited Retaliation | Can retaliate unlimited times |
| Regeneration | Heals at start of each round |
| Undead | Immune to morale, mind spells, can't be resurrected |
| Jousting | Bonus damage based on distance traveled |
| Double Strike | Attacks twice per turn |
| Fire Shield | Damages melee attackers |
| Lightning Strike | Returns to position after attack |
| Aging | Reduces target max HP |
| Death Stare | Chance to instantly kill |
| Blind | Chance to blind target |
| Paralyze | Chance to stun target |
| Magic Immunity | Immune to all spells |
| Magic Resistance | % chance to resist spells |
| Ranged | Can attack from distance |
| No Obstacle Penalty | Ranged ignores wall penalty |
| Spellcaster | Can cast spells |

### Army Stack Limits

- Maximum **7 creature stacks** per hero
- Each stack can contain **unlimited creatures** of same type
- **Split/merge** stacks freely in army management
- **Creatures fight as stack** (combined health pool)

### Morale System

Morale affects combat initiative:

| Morale Level | Effect |
|--------------|--------|
| +3 (Excellent) | 12.5% chance for bonus turn |
| +2 (Good) | 8.3% chance for bonus turn |
| +1 (High) | 4.2% chance for bonus turn |
| 0 (Neutral) | No effect |
| -1 (Low) | 4.2% chance to freeze |
| -2 (Poor) | 8.3% chance to freeze |
| -3 (Terrible) | 12.5% chance to freeze |

**Morale Modifiers:**
- Same faction alignment: +1
- Mixed factions: -1 per extra faction
- Undead in army: -1 (living creatures)
- Artifacts/spells: Variable
- Leadership skill: +1/2/3

### Luck System

| Luck Level | Effect |
|------------|--------|
| +3 | 12.5% chance for double damage |
| +2 | 8.3% chance for double damage |
| +1 | 4.2% chance for double damage |
| 0 | No effect |
| -1 | 4.2% chance for half damage |
| -2 | 8.3% chance for half damage |
| -3 | 12.5% chance for half damage |

---

## Combat System

### Hex-Grid Battlefield

```
┌─────────────────────────────────────────────────────────────┐
│                    BATTLEFIELD LAYOUT                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    15 hexes wide × 11 hexes tall                            │
│                                                             │
│    ATTACKER SIDE          │          DEFENDER SIDE          │
│    (Left)                 │          (Right)                │
│                           │                                 │
│    ╔═══╗                  │                 ╔═══╗           │
│    ║ 1 ║                  │    SIEGE        ║ 1 ║           │
│    ╠═══╣                  │    WALLS        ╠═══╣           │
│    ║ 2 ║                  │    ════════     ║ 2 ║           │
│    ╠═══╣                  │    │      │     ╠═══╣           │
│    ║ 3 ║                  │    │TOWER │     ║ 3 ║           │
│    ╠═══╣                  │    │      │     ╠═══╣           │
│    ║ 4 ║                  │    ════════     ║ 4 ║           │
│    ╠═══╣                  │                 ╠═══╣           │
│    ║ 5 ║                  │                 ║ 5 ║           │
│    ╠═══╣                  │                 ╠═══╣           │
│    ║ 6 ║                  │                 ║ 6 ║           │
│    ╠═══╣                  │                 ╠═══╣           │
│    ║ 7 ║                  │                 ║ 7 ║           │
│    ╚═══╝                  │                 ╚═══╝           │
│                           │                                 │
│    (Deployment zones)                                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Combat Turn Order

1. Calculate **initiative** for all stacks: `speed × (1 + speedModifiers)`
2. Sort stacks by initiative (highest first)
3. On tie: **Defender advantage** (defender acts first)
4. Each stack gets **one action** per round:
   - Move
   - Attack (melee or ranged)
   - Wait (delay until end of round)
   - Defend (+2 defense until next turn)
   - Cast spell (hero action)
   - Use ability

### Damage Calculation

```javascript
// Base damage formula
function calculateDamage(attacker, defender, attackerCount) {
  // 1. Base damage
  const baseDamage = randomInRange(attacker.damage.min, attacker.damage.max);
  
  // 2. Attack vs Defense modifier
  const statDiff = attacker.attack - defender.defense;
  let modifier;
  
  if (statDiff >= 0) {
    // Attacker advantage: +5% per point, max +300%
    modifier = 1 + Math.min(statDiff * 0.05, 3.0);
  } else {
    // Defender advantage: -2.5% per point, max -70%
    modifier = Math.max(1 + statDiff * 0.025, 0.3);
  }
  
  // 3. Stack size
  const totalDamage = baseDamage * attackerCount * modifier;
  
  // 4. Apply bonuses (hero attack, offense skill, artifacts)
  const bonusMultiplier = 1 + (heroAttack * 0.05) + offenseBonus + artifactBonus;
  
  // 5. Apply reductions (hero defense, armorer skill)
  const reductionMultiplier = 1 - (heroDefense * 0.025) - armorerBonus;
  
  return Math.round(totalDamage * bonusMultiplier * reductionMultiplier);
}
```

### Casualties Calculation

```javascript
function calculateCasualties(damage, defender, stackSize) {
  const healthPerUnit = defender.health;
  const totalHealth = healthPerUnit * stackSize + defender.currentDamage;
  
  const remainingHealth = totalHealth - damage;
  
  if (remainingHealth <= 0) {
    return { dead: stackSize, remainingHealth: 0 };
  }
  
  const dead = Math.floor((totalHealth - remainingHealth) / healthPerUnit);
  const leftoverDamage = (totalHealth - remainingHealth) % healthPerUnit;
  
  return { dead, remainingHealth: healthPerUnit - leftoverDamage };
}
```

### Siege Combat

When attacking a town with fortifications:

| Fortification | Effect |
|---------------|--------|
| Fort | Basic walls, moat |
| Citadel | Stronger walls, 1 tower |
| Castle | Strongest walls, 2 towers, arrow slits |

**Siege Mechanics:**
- Walls block movement (must destroy or climb)
- Towers fire at attackers each round
- Catapult (attacker) destroys wall sections
- Ballista (defender) targets attacker stacks
- Moat slows attackers
- Drawbridge can be raised/lowered

### Combat Resolution

**Victory Conditions:**
- All enemy stacks destroyed
- Enemy hero flees/surrenders
- Auto-combat calculation

**Defeat Consequences:**
- Hero loses army
- Hero captured (can be rescued from prison)
- Hero may escape (based on skills/artifacts)

**Experience Gain:**
```javascript
expGained = enemyArmyValue * (enemyCasualties / totalEnemyArmy) * difficultyModifier
```

---

## Magic System

### Spell Schools

| School | Theme | Primary Effects |
|--------|-------|-----------------|
| Air | Speed, precision | Haste, precision, chain lightning |
| Earth | Defense, slow | Slow, shield, meteor shower |
| Fire | Damage, aggression | Fireball, armageddon, berserk |
| Water | Healing, utility | Cure, clone, prayer |

### Spell Tiers

| Tier | Mana Guild Level | Example Spells |
|------|------------------|----------------|
| 1 | 1 | Magic Arrow, Haste, Shield, Bless |
| 2 | 2 | Lightning Bolt, Ice Bolt, Cure, Weakness |
| 3 | 3 | Fireball, Frost Ring, Animate Dead, Forgetfulness |
| 4 | 4 | Meteor Shower, Chain Lightning, Resurrection, Berserk |
| 5 | 5 | Armageddon, Implosion, Summon Elemental, Dimension Door |

### Spell Data Structure

```javascript
SPELLS = {
  magicArrow: {
    school: "all", // Available to all schools
    tier: 1,
    manaCost: 5,
    type: "damage",
    target: "single",
    effect: (caster) => 10 + caster.power * 10,
    description: "Deals (10 + Power×10) damage to target"
  },
  
  haste: {
    school: "air",
    tier: 1,
    manaCost: 6,
    type: "buff",
    target: "single",
    effect: {
      basic: { speed: 3 },
      advanced: { speed: 5 },
      expert: { speed: 5, target: "all_friendly" }
    },
    duration: "combat",
    description: "Increases speed of target stack"
  },
  
  armageddon: {
    school: "fire",
    tier: 5,
    manaCost: 24,
    type: "damage",
    target: "all",
    effect: (caster) => 30 + caster.power * 50,
    description: "Damages ALL creatures on battlefield"
  },
  
  resurrection: {
    school: "water",
    tier: 4,
    manaCost: 20,
    type: "resurrection",
    target: "single",
    effect: (caster, skill) => {
      const base = 40 + caster.power * 50;
      if (skill === "expert") return base; // Permanent
      return base; // Temporary (die after battle)
    }
  }
}
```

### Adventure Map Spells

| Spell | Effect | Mana |
|-------|--------|------|
| View Air | Reveal resources | 2 |
| View Earth | Reveal terrain | 2 |
| Town Portal | Teleport to town | 16 |
| Dimension Door | Teleport on map | 25 |
| Fly | Cross any terrain | 20 |
| Water Walk | Cross water | 12 |
| Summon Boat | Create boat at shore | 8 |
| Scuttle Boat | Destroy boat | 8 |

### Mana Management

```javascript
// Mana pool
maxMana = hero.knowledge * 10 * (1 + intelligenceBonus);

// Daily regeneration
manaRegen = baseManaRegen + mysticismBonus;

// Spell cost reduction
actualCost = spellBaseCost * (1 - costReductionModifiers);
```

---

## Resource Economy

### Resource Types

| Resource | Primary Use | Rarity |
|----------|-------------|--------|
| Gold | Everything | Common |
| Wood | Buildings, tier 1-3 creatures | Common |
| Ore | Buildings, tier 1-3 creatures | Common |
| Mercury | Magic buildings, tier 5-7 | Rare |
| Sulfur | Fire/demon creatures, siege | Rare |
| Crystal | Magic buildings, elementals | Rare |
| Gems | Angel/fairy creatures, artifacts | Rare |

### Resource Acquisition

| Source | Frequency | Amount |
|--------|-----------|--------|
| Mines | Daily | Varies by type |
| Town Hall | Daily | 500-4000 gold |
| Markets | On trade | Variable |
| Map pickups | One-time | Variable |
| Quest rewards | Quest completion | Variable |
| Battle loot | After combat | % of enemy value |

### Market Trading

```javascript
MARKET_RATES = {
  base: {
    wood: { gold: 100 },
    ore: { gold: 100 },
    mercury: { gold: 250 },
    sulfur: { gold: 250 },
    crystal: { gold: 250 },
    gems: { gold: 250 }
  },
  
  // Trading between resources
  // Rate improves with more marketplaces
  getTradeRate: (marketplaceCount) => {
    const rates = [10, 7, 5, 4, 3, 2.5, 2, 1.5, 1]; // 1:10 down to 1:1
    return rates[Math.min(marketplaceCount, 8)];
  }
}
```

---

## Victory Conditions

### Standard Victory Types

| Condition | Description |
|-----------|-------------|
| Defeat All Enemies | Eliminate all opponent heroes and towns |
| Capture Town | Control a specific town |
| Defeat Hero | Defeat a specific enemy hero |
| Find Artifact | Locate and collect a specific artifact |
| Accumulate Resources | Gather X amount of specific resource |
| Accumulate Creatures | Recruit X creatures of specific type |
| Upgrade Town | Build specific structure in town |
| Puzzle Map | Collect all obelisk pieces, find grail |
| Time Limit | Survive X months |
| Transport Artifact | Bring artifact to specific location |

### Loss Conditions

| Condition | Description |
|-----------|-------------|
| Lose All Heroes/Towns | Standard defeat |
| Lose Specific Town | Must not lose this town |
| Lose Specific Hero | Must keep this hero alive |
| Time Limit Exceeded | Did not win in time |

---

## Technical Architecture

### Module Structure

```
src/
├── components/
│   ├── adventure/
│   │   ├── AdventureMap.jsx
│   │   ├── MapTile.jsx
│   │   ├── MapObject.jsx
│   │   ├── FogOfWar.jsx
│   │   ├── HeroSprite.jsx
│   │   └── MiniMap.jsx
│   │
│   ├── combat/
│   │   ├── BattleScreen.jsx
│   │   ├── HexGrid.jsx
│   │   ├── HexTile.jsx
│   │   ├── CreatureStack.jsx
│   │   ├── SpellEffect.jsx
│   │   ├── DamageNumber.jsx
│   │   └── CombatUI.jsx
│   │
│   ├── town/
│   │   ├── TownScreen.jsx
│   │   ├── BuildingSlot.jsx
│   │   ├── DwellingPanel.jsx
│   │   ├── RecruitmentDialog.jsx
│   │   ├── MageGuild.jsx
│   │   └── Marketplace.jsx
│   │
│   ├── hero/
│   │   ├── HeroScreen.jsx
│   │   ├── ArmyPanel.jsx
│   │   ├── SkillTree.jsx
│   │   ├── ArtifactSlots.jsx
│   │   ├── SpellBook.jsx
│   │   └── ExperienceBar.jsx
│   │
│   ├── ui/
│   │   ├── ResourceBar.jsx
│   │   ├── TurnIndicator.jsx
│   │   ├── ActionPanel.jsx
│   │   ├── Dialog.jsx
│   │   ├── Tooltip.jsx
│   │   └── Button.jsx
│   │
│   └── svg/
│       ├── TerrainSVG.jsx
│       ├── CreatureSVG.jsx
│       ├── BuildingSVG.jsx
│       ├── IconSVG.jsx
│       └── EffectSVG.jsx
│
├── hooks/
│   ├── useGameState.js
│   ├── useCombat.js
│   ├── useMovement.js
│   ├── useAI.js
│   └── useAnimation.js
│
├── state/
│   ├── gameSlice.js
│   ├── mapSlice.js
│   ├── heroSlice.js
│   ├── combatSlice.js
│   └── store.js
│
├── engine/
│   ├── pathfinding.js
│   ├── combatResolver.js
│   ├── damageCalculator.js
│   ├── aiController.js
│   └── mapGenerator.js
│
├── data/
│   ├── creatures.json
│   ├── spells.json
│   ├── artifacts.json
│   ├── buildings.json
│   ├── skills.json
│   └── terrains.json
│
└── utils/
    ├── hexMath.js
    ├── random.js
    ├── animations.js
    └── constants.js
```

### Technology Stack

| Layer | Technology |
|-------|------------|
| UI Framework | React 18+ |
| State Management | Zustand or Redux Toolkit |
| Graphics | SVG (inline React components) |
| Styling | Tailwind CSS + CSS Variables |
| Animation | Framer Motion / CSS Animations |
| Sound | Howler.js |
| Pathfinding | A* Algorithm (custom) |
| Build Tool | Vite |
| Testing | Vitest + React Testing Library |

---

## Component Specifications

### AdventureMap Component

```jsx
/**
 * AdventureMap - Main adventure map viewport
 * 
 * Props:
 * @param {Object} mapData - Terrain and object data
 * @param {Object} viewport - Camera position and zoom
 * @param {Object} selectedHero - Currently selected hero
 * @param {Function} onTileClick - Tile interaction handler
 * @param {Function} onHeroMove - Hero movement handler
 * 
 * Features:
 * - Panning and zooming
 * - Tile-based rendering with culling
 * - Fog of war overlay
 * - Path preview on hover
 * - Object interaction
 */
const AdventureMap = ({ mapData, viewport, selectedHero, onTileClick, onHeroMove }) => {
  // Render visible tiles only (viewport culling)
  // Handle mouse/touch input for panning
  // Display path preview
  // Manage fog of war state
}
```

### HexGrid Combat Component

```jsx
/**
 * HexGrid - Tactical combat battlefield
 * 
 * Props:
 * @param {number} width - Grid width in hexes (15)
 * @param {number} height - Grid height in hexes (11)
 * @param {Array} obstacles - Obstacle positions
 * @param {Array} stacks - Creature stacks on battlefield
 * @param {Object} activeStack - Currently acting stack
 * @param {Function} onHexClick - Hex selection handler
 * @param {Function} onStackClick - Stack targeting handler
 * 
 * Features:
 * - Offset hex coordinate system
 * - Movement range highlighting
 * - Attack range visualization
 * - Spell targeting overlay
 * - Animation support for actions
 */
const HexGrid = ({ width, height, obstacles, stacks, activeStack, onHexClick, onStackClick }) => {
  // Calculate valid move/attack hexes
  // Render hex tiles with appropriate states
  // Handle stack selection and targeting
  // Display action animations
}
```

### TownScreen Component

```jsx
/**
 * TownScreen - Town building and management interface
 * 
 * Props:
 * @param {Object} town - Town data
 * @param {Object} player - Current player resources
 * @param {Function} onBuild - Building construction handler
 * @param {Function} onRecruit - Creature recruitment handler
 * @param {Function} onExit - Exit town handler
 * 
 * Features:
 * - Faction-themed visual design
 * - Building tree visualization
 * - Creature dwelling management
 * - Resource cost preview
 * - Build queue management
 */
const TownScreen = ({ town, player, onBuild, onRecruit, onExit }) => {
  // Display town background for faction
  // Render building slots with current buildings
  // Show available upgrades
  // Handle recruitment dialog
  // Display garrison
}
```

### CreatureStack Component

```jsx
/**
 * CreatureStack - Battlefield creature display
 * 
 * Props:
 * @param {Object} stack - Stack data (creature, count, health)
 * @param {Object} position - Hex position
 * @param {boolean} isActive - Currently acting
 * @param {boolean} isSelected - Selected for targeting
 * @param {string} animation - Current animation state
 * @param {Function} onClick - Click handler
 * 
 * Features:
 * - Creature SVG sprite
 * - Stack count display
 * - Health bar
 * - Status effect icons
 * - Animation states (idle, attack, defend, hit, death)
 */
const CreatureStack = ({ stack, position, isActive, isSelected, animation, onClick }) => {
  // Render creature sprite
  // Display count badge
  // Show health bar
  // Apply animation class
  // Handle click for targeting
}
```

---

## SVG Graphics System

### Terrain Tiles

```jsx
/**
 * TerrainTile - Modular terrain SVG component
 * 
 * Terrain types rendered as SVG paths with consistent style:
 * - Grass: Green with subtle texture
 * - Water: Blue with wave pattern
 * - Mountain: Gray with peaks
 * - Forest: Green with tree shapes
 * - Desert: Tan with dune patterns
 * - Swamp: Dark green with murky water
 * - Lava: Orange/red with glow
 * - Snow: White with sparkle
 */
const TerrainTile = ({ type, x, y, size }) => {
  const terrainStyles = {
    grass: {
      fill: '#4a7c20',
      pattern: 'grassTexture'
    },
    water: {
      fill: '#2563eb',
      pattern: 'wavePattern',
      animated: true
    },
    // ... more terrain types
  };
  
  return (
    <g transform={`translate(${x}, ${y})`}>
      <polygon 
        points={hexPoints(size)} 
        fill={terrainStyles[type].fill}
      />
      {/* Texture overlay */}
      {/* Edge blending */}
    </g>
  );
};
```

### Creature Sprites

Each creature type has an SVG sprite with multiple poses:

```jsx
/**
 * CreatureSprite - Animated creature SVG
 * 
 * Poses:
 * - idle: Default standing animation
 * - walk: Movement animation
 * - attack: Attack animation frames
 * - defend: Defensive stance
 * - hit: Taking damage
 * - death: Death animation
 * - special: Special ability animation
 */
const CreatureSprite = ({ creature, pose, facing, scale }) => {
  // Creature SVG data stored per type
  // Use CSS transforms for facing direction
  // Apply animation keyframes for pose
  
  return (
    <svg viewBox="0 0 100 100" style={{ transform: `scaleX(${facing === 'left' ? -1 : 1})` }}>
      <g className={`creature-${pose}`}>
        {creatureData[creature].paths.map((path, i) => (
          <path key={i} d={path.d} fill={path.fill} />
        ))}
      </g>
    </svg>
  );
};
```

### Building Graphics

```jsx
/**
 * BuildingSVG - Town building graphics
 * 
 * Each building has:
 * - Base structure
 * - Faction-specific styling
 * - Construction animation
 * - Active/inactive states
 */
const BuildingSVG = ({ building, faction, constructed, level }) => {
  const buildingData = getBuildingData(building, faction);
  
  return (
    <svg viewBox={buildingData.viewBox}>
      {/* Foundation */}
      <rect ... />
      
      {/* Main structure */}
      {buildingData.paths.map((path, i) => (
        <path 
          key={i} 
          d={path.d} 
          fill={constructed ? path.fill : '#666'}
          opacity={constructed ? 1 : 0.5}
        />
      ))}
      
      {/* Level indicator */}
      {level > 1 && <LevelBadge level={level} />}
      
      {/* Construction overlay */}
      {!constructed && <ConstructionOverlay />}
    </svg>
  );
};
```

### Icon System

```jsx
/**
 * IconSVG - Unified icon component
 * 
 * Categories:
 * - resources: gold, wood, ore, mercury, sulfur, crystal, gems
 * - stats: attack, defense, power, knowledge, speed
 * - skills: all 28 secondary skills
 * - spells: spell school icons
 * - ui: arrows, buttons, indicators
 */
const IconSVG = ({ icon, size = 24, color }) => {
  const iconPaths = {
    gold: 'M12 2C6.48 2 2 6.48 2 12s4.48 10...',
    attack: 'M14.5 2L19 6.5 6.5 19 2 14.5z...',
    // ... all icons
  };
  
  return (
    <svg width={size} height={size} viewBox="0 0 24 24">
      <path d={iconPaths[icon]} fill={color || 'currentColor'} />
    </svg>
  );
};
```

### Effect Animations

```jsx
/**
 * SpellEffect - Animated spell effects
 * 
 * Effect types:
 * - projectile: Travels from caster to target
 * - explosion: Expands from center
 * - aura: Persistent around target
 * - beam: Line between points
 * - summon: Appears at location
 */
const SpellEffect = ({ type, start, end, onComplete }) => {
  const [progress, setProgress] = useState(0);
  
  useEffect(() => {
    // Animate effect based on type
    // Call onComplete when finished
  }, []);
  
  return (
    <g className="spell-effect">
      {type === 'fireball' && (
        <FireballEffect progress={progress} start={start} end={end} />
      )}
      {type === 'lightning' && (
        <LightningEffect from={start} to={end} />
      )}
      {/* More effect types */}
    </g>
  );
};
```

---

## Data Models

### Game State

```javascript
const gameState = {
  // Meta
  id: "game-uuid",
  turn: 1,
  day: 1,
  week: 1,
  month: 1,
  currentPlayer: 0,
  phase: "action", // "upkeep" | "action" | "combat" | "end"
  
  // Players
  players: [
    {
      id: "player-1",
      name: "Player 1",
      faction: "castle",
      color: "#ff0000",
      resources: {
        gold: 10000,
        wood: 20,
        ore: 20,
        mercury: 5,
        sulfur: 5,
        crystal: 5,
        gems: 5
      },
      heroes: ["hero-1", "hero-2"],
      towns: ["town-1"],
      defeated: false
    }
  ],
  
  // Map
  map: {
    width: 144,
    height: 144,
    terrain: [], // 2D array of terrain types
    objects: [], // Map objects
    fog: [], // Visibility per player
  },
  
  // Entities
  heroes: {}, // Hero objects by ID
  towns: {}, // Town objects by ID
  creatures: {} // Roaming creature armies
};
```

### Hero Model

```javascript
const hero = {
  id: "hero-1",
  name: "Sir Roland",
  class: "knight",
  level: 1,
  experience: 0,
  owner: "player-1",
  
  // Position
  position: { x: 10, y: 15 },
  movementRemaining: 1500,
  
  // Stats
  stats: {
    attack: 2,
    defense: 2,
    power: 1,
    knowledge: 1
  },
  
  // Skills (max 8)
  skills: [
    { skill: "leadership", level: "basic" },
    { skill: "tactics", level: "advanced" }
  ],
  
  // Spells known
  spells: ["haste", "bless", "magicArrow"],
  mana: 10,
  maxMana: 10,
  
  // Army (max 7 stacks)
  army: [
    { creature: "pikeman", count: 20 },
    { creature: "archer", count: 15 },
    { creature: "griffin", count: 8 },
    null, null, null, null
  ],
  
  // Equipment
  artifacts: {
    head: null,
    neck: null,
    torso: "armorOfWonder",
    leftHand: "dragonShield",
    rightHand: "swordOfHellfire",
    leftRing: null,
    rightRing: "ringOfVitality",
    feet: "bootsOfSpeed",
    misc: [null, null, null, null, null]
  }
};
```

### Town Model

```javascript
const town = {
  id: "town-1",
  name: "Steadwick",
  faction: "castle",
  owner: "player-1",
  position: { x: 20, y: 25 },
  
  // Buildings (boolean or level)
  buildings: {
    townHall: 3, // 1=town hall, 2=city hall, 3=capitol
    fort: 2, // 1=fort, 2=citadel, 3=castle
    tavern: true,
    blacksmith: true,
    marketplace: true,
    mageGuild: 3, // Level 1-5
    
    // Dwellings
    dwelling1: true, // Guardhouse
    dwelling1Up: true, // Upgraded
    dwelling2: true,
    dwelling2Up: false,
    dwelling3: true,
    dwelling3Up: true,
    dwelling4: true,
    dwelling4Up: false,
    dwelling5: false,
    dwelling5Up: false,
    dwelling6: false,
    dwelling6Up: false,
    dwelling7: false,
    dwelling7Up: false,
    
    // Special buildings
    grail: false, // If grail structure built here
  },
  
  // Creature pool (refreshes weekly)
  availableCreatures: {
    pikeman: 14,
    halberdier: 0,
    archer: 9,
    marksman: 0,
    griffin: 7,
    royalGriffin: 0,
    // ...
  },
  
  // Garrison
  garrison: [
    { creature: "pikeman", count: 50 },
    null, null, null, null, null, null
  ],
  
  // Visiting hero
  visitingHero: null
};
```

### Combat State

```javascript
const combatState = {
  id: "combat-uuid",
  type: "field", // "field" | "siege" | "boat"
  
  attacker: {
    heroId: "hero-1",
    stacks: [
      {
        id: "stack-1",
        creature: "champion",
        count: 15,
        currentHealth: 150, // Per creature
        position: { q: 0, r: 5 },
        hasActed: false,
        effects: [{ type: "haste", duration: 2 }]
      }
      // ...
    ]
  },
  
  defender: {
    heroId: "hero-2", // or null for monster
    stacks: [/* ... */],
    fortification: 2 // Siege: 1=fort, 2=citadel, 3=castle
  },
  
  battlefield: {
    width: 15,
    height: 11,
    obstacles: [
      { position: { q: 7, r: 5 }, type: "rock" }
    ],
    walls: [/* Siege wall positions */],
    towers: [/* Tower positions */]
  },
  
  turnOrder: ["stack-1", "stack-5", "stack-2", /* ... */],
  currentTurn: 0,
  round: 1,
  
  log: [
    { type: "attack", attacker: "stack-1", defender: "stack-5", damage: 450 },
    { type: "spell", caster: "hero-1", spell: "fireball", targets: ["stack-6"], damage: 280 }
  ]
};
```

---

## State Management

### Store Structure (Zustand)

```javascript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

const useGameStore = create(
  devtools(
    persist(
      (set, get) => ({
        // Game state
        game: null,
        
        // UI state
        ui: {
          screen: 'adventure', // 'adventure' | 'town' | 'hero' | 'combat' | 'menu'
          selectedHero: null,
          hoveredTile: null,
          modal: null,
        },
        
        // Actions
        initGame: (config) => {
          const game = generateGame(config);
          set({ game });
        },
        
        moveHero: (heroId, path) => {
          set((state) => ({
            game: {
              ...state.game,
              heroes: {
                ...state.game.heroes,
                [heroId]: moveHeroAlongPath(state.game.heroes[heroId], path)
              }
            }
          }));
        },
        
        startCombat: (attackerId, defenderId) => {
          const combat = initializeCombat(
            get().game.heroes[attackerId],
            get().game.heroes[defenderId] || get().game.creatures[defenderId]
          );
          set((state) => ({
            game: { ...state.game, combat },
            ui: { ...state.ui, screen: 'combat' }
          }));
        },
        
        executeCombatAction: (action) => {
          set((state) => ({
            game: {
              ...state.game,
              combat: processCombatAction(state.game.combat, action)
            }
          }));
        },
        
        buildStructure: (townId, building) => {
          set((state) => ({
            game: {
              ...state.game,
              towns: {
                ...state.game.towns,
                [townId]: addBuilding(state.game.towns[townId], building)
              },
              players: deductResources(state.game.players, building.cost)
            }
          }));
        },
        
        endTurn: () => {
          set((state) => {
            const nextState = processEndTurn(state.game);
            return { game: nextState };
          });
        }
      }),
      {
        name: 'homm3-storage',
        partialize: (state) => ({ game: state.game })
      }
    )
  )
);
```

### Custom Hooks

```javascript
// Combat logic hook
const useCombat = () => {
  const { combat, executeCombatAction } = useGameStore();
  
  const getValidMoves = useCallback((stackId) => {
    // Calculate hexes the stack can reach
    return calculateMoveRange(combat, stackId);
  }, [combat]);
  
  const getValidAttacks = useCallback((stackId) => {
    // Calculate valid attack targets
    return calculateAttackTargets(combat, stackId);
  }, [combat]);
  
  const attack = useCallback((attackerId, defenderId) => {
    executeCombatAction({
      type: 'attack',
      attacker: attackerId,
      defender: defenderId
    });
  }, [executeCombatAction]);
  
  const castSpell = useCallback((spellId, targets) => {
    executeCombatAction({
      type: 'spell',
      spell: spellId,
      targets
    });
  }, [executeCombatAction]);
  
  return { combat, getValidMoves, getValidAttacks, attack, castSpell };
};

// Pathfinding hook
const usePathfinding = () => {
  const { map, heroes } = useGameStore((state) => state.game);
  
  const findPath = useCallback((heroId, destination) => {
    const hero = heroes[heroId];
    return aStarPath(map, hero.position, destination, hero);
  }, [map, heroes]);
  
  const getMovementCost = useCallback((path) => {
    return path.reduce((cost, tile) => {
      return cost + getTerrainCost(map.terrain[tile.y][tile.x]);
    }, 0);
  }, [map]);
  
  return { findPath, getMovementCost };
};
```

---

## Accessibility & Performance

### Accessibility Requirements

| Feature | Implementation |
|---------|----------------|
| Keyboard Navigation | Full keyboard support for all actions |
| Screen Reader | ARIA labels on all interactive elements |
| Color Blind Mode | Alternative color schemes, patterns |
| Text Scaling | Responsive font sizing |
| Reduced Motion | Option to disable animations |
| Focus Indicators | Clear visual focus states |
| Tooltips | Descriptive tooltips on all elements |

### Keyboard Controls

| Key | Adventure Map | Combat | Town |
|-----|---------------|--------|------|
| Arrow Keys | Pan map | Move selection | Navigate buildings |
| Space | Select/Confirm | Confirm action | Build |
| Enter | Interact | Execute attack | Confirm |
| Escape | Cancel/Menu | Cancel action | Exit |
| H | Cycle heroes | — | — |
| T | End turn | — | — |
| M | Open minimap | — | Marketplace |
| S | — | Cast spell | — |
| D | — | Defend | — |
| W | — | Wait | — |

### Performance Targets

| Metric | Target |
|--------|--------|
| Initial Load | < 3 seconds |
| Time to Interactive | < 5 seconds |
| Frame Rate | 60 FPS |
| Memory Usage | < 200 MB |
| Map Render | < 16ms per frame |
| Combat Animation | 60 FPS smooth |

### Optimization Strategies

```javascript
// Viewport culling for adventure map
const renderVisibleTiles = (viewport, tiles) => {
  const { x, y, width, height } = viewport;
  const margin = 2; // Extra tiles for smooth scrolling
  
  return tiles.filter(tile => 
    tile.x >= x - margin && 
    tile.x <= x + width + margin &&
    tile.y >= y - margin &&
    tile.y <= y + height + margin
  );
};

// Memoized components
const MemoizedMapTile = React.memo(MapTile, (prev, next) => 
  prev.terrain === next.terrain && 
  prev.visibility === next.visibility &&
  prev.x === next.x && 
  prev.y === next.y
);

// Web Worker for pathfinding
const pathfindingWorker = new Worker('./pathfindingWorker.js');

const asyncFindPath = (start, end, map) => {
  return new Promise((resolve) => {
    pathfindingWorker.postMessage({ start, end, map });
    pathfindingWorker.onmessage = (e) => resolve(e.data.path);
  });
};

// RequestAnimationFrame for smooth animations
const useAnimationFrame = (callback) => {
  const requestRef = useRef();
  
  useEffect(() => {
    const animate = (time) => {
      callback(time);
      requestRef.current = requestAnimationFrame(animate);
    };
    requestRef.current = requestAnimationFrame(animate);
    return () => cancelAnimationFrame(requestRef.current);
  }, [callback]);
};
```

---

## Testing Scenarios

### Unit Tests

| Component | Test Cases |
|-----------|------------|
| Damage Calculator | Attack vs defense scenarios, bonuses |
| Pathfinding | Shortest path, obstacles, terrain costs |
| Movement | Points deduction, terrain modifiers |
| Combat Order | Initiative sorting, ties |
| Spell Effects | Damage, duration, stacking |

### Integration Tests

| Scenario | Validation |
|----------|------------|
| Full Combat | Start to finish battle resolution |
| Town Building | Dependencies, resources, effects |
| Hero Leveling | Experience, skill selection |
| Map Exploration | Fog of war, object interaction |
| AI Turn | Valid moves, combat decisions |

### E2E Tests

| Flow | Steps |
|------|-------|
| New Game | Create game → Place starting units → First turn |
| Win Condition | Play until victory/defeat |
| Combat Flow | Initiate → Fight → Resolve → Return to map |
| Town Management | Enter → Build → Recruit → Exit |

---

## Extended Features

### Multiplayer Support

- **Hot-Seat**: Pass-and-play on single device
- **Async**: Turn-based with notifications
- **Real-time**: Live combat (optional)

### AI System

| Difficulty | Behavior |
|------------|----------|
| Easy | Random moves, no optimization |
| Normal | Basic strategy, some optimization |
| Hard | Optimal plays, resource management |
| Expert | Cheating (bonus resources), perfect play |

### Map Editor

- Terrain painting
- Object placement
- Victory condition setup
- Player starting positions
- Import/export maps

### Campaign Mode

- Linked scenarios
- Persistent heroes
- Story elements
- Branching paths

---

## Appendix: Hex Math Utilities

```javascript
// Offset coordinates (odd-q layout)
const HexMath = {
  // Pixel to hex
  pixelToHex: (x, y, size) => {
    const q = (2/3 * x) / size;
    const r = (-1/3 * x + Math.sqrt(3)/3 * y) / size;
    return cubeToOffset(cubeRound({ x: q, y: -q-r, z: r }));
  },
  
  // Hex to pixel
  hexToPixel: (q, r, size) => {
    const x = size * 3/2 * q;
    const y = size * Math.sqrt(3) * (r + 0.5 * (q & 1));
    return { x, y };
  },
  
  // Get hex neighbors
  neighbors: (q, r) => {
    const directions = (q & 1) === 0 
      ? [
          [1, 0], [1, -1], [0, -1],
          [-1, -1], [-1, 0], [0, 1]
        ]
      : [
          [1, 1], [1, 0], [0, -1],
          [-1, 0], [-1, 1], [0, 1]
        ];
    return directions.map(([dq, dr]) => ({ q: q + dq, r: r + dr }));
  },
  
  // Distance between hexes
  distance: (a, b) => {
    const ac = offsetToCube(a);
    const bc = offsetToCube(b);
    return (Math.abs(ac.x - bc.x) + Math.abs(ac.y - bc.y) + Math.abs(ac.z - bc.z)) / 2;
  },
  
  // Hexes in range
  hexesInRange: (center, range) => {
    const results = [];
    for (let dq = -range; dq <= range; dq++) {
      for (let dr = Math.max(-range, -dq - range); dr <= Math.min(range, -dq + range); dr++) {
        results.push({ q: center.q + dq, r: center.r + dr });
      }
    }
    return results;
  },
  
  // Line between hexes
  lineDraw: (a, b) => {
    const N = HexMath.distance(a, b);
    const results = [];
    for (let i = 0; i <= N; i++) {
      results.push(cubeRound(cubeLerp(offsetToCube(a), offsetToCube(b), i / N)));
    }
    return results.map(cubeToOffset);
  }
};
```

---

*Document Version: 1.0*  
*Specification Status: Complete*  
*Ready for Implementation: ✓*
