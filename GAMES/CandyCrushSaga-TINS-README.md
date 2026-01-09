# Candy Crush Saga Clone

A feature-complete match-3 puzzle game built in React, replicating the core gameplay mechanics, special candy systems, level objectives, blockers, and polished "juicy" feedback of King's Candy Crush Saga. This implementation focuses on delivering satisfying cascading chain reactions, strategic special candy combinations, and progressively challenging levels.

The game operates on a grid-based board where players swap adjacent candies to create matches of 3 or more identical colors. Larger matches create special candies with powerful effects. Multiple level objective types (jelly clearing, ingredient drops, candy orders) combined with diverse blockers create strategic depth. Every interaction rewards players with layered audio-visual feedback that escalates with combo size.

---

## Functionality

### Core Match-3 Engine

#### Board Configuration
- Default board size: 9 columns × 9 rows (81 cells maximum)
- Boards can have irregular shapes with blocked/missing cells
- 6 candy colors: Blue, Green, Orange, Purple, Red, Yellow
- Each candy is a distinct visual entity with color and position

#### Swap Mechanics
- Players can only swap orthogonally adjacent candies (not diagonal)
- A swap is only valid if it creates at least one match
- Invalid swaps trigger a "ping-pong" animation (0.25s) returning candies to original positions
- Valid swaps animate candies moving to each other's positions (0.15s)

#### Match Detection
- Minimum match: 3 identical candies in horizontal OR vertical line
- Diagonal matches are never valid
- Match detection runs after every swap and after every cascade settle
- Algorithm scans all rows horizontally, then all columns vertically
- Overlapping matches (L-shapes, T-shapes, crosses) are detected as single match events
- Match shapes determine special candy creation (see Special Candies section)

#### Cascade System (The Core Loop)
After any match is detected, the following sequence executes:

1. **Match Phase** (0.0s - 0.1s)
   - Matched candies play destruction animation (scale to 0 with particles)
   - Score awarded for match (base 60 points for 3-match)
   - Cascade counter increments if this isn't the first match

2. **Gravity Phase** (0.1s - 0.3s)
   - Empty spaces detected column by column
   - Candies above empty spaces fall down
   - Fall uses eased acceleration (not linear), with slight bounce on landing

3. **Spawn Phase** (0.3s - 0.5s)
   - New candies spawn at top of each column with empty spaces
   - Candies enter with slight scale-up animation
   - Color selection uses weighted randomness (prevents clustering)

4. **Re-check Phase** (0.5s+)
   - Full board scanned for new matches
   - If matches found: return to step 1 (cascade continues)
   - If no matches: return control to player

5. **No Moves Check**
   - If no valid moves exist anywhere on board, trigger automatic shuffle
   - Shuffle preserves color distribution, only rearranges positions
   - Shuffle excludes: special candies, blockers, ingredients
   - Shuffle guarantees at least one valid move exists

#### Cascade Scoring
- Base 3-match: 60 points
- 4-match: 120 points
- 5-match: 200 points
- Cascade bonus: +120 points × cascade level (1st cascade +120, 2nd +240, 3rd +360...)
- Special candy creation bonus: +500 points
- Special candy activation bonus: varies by type

---

### Special Candies

Special candies are created by matching more than 3 candies in specific patterns. They have powerful effects when activated (matched or triggered by other specials).

#### Striped Candy
- **Creation**: Match exactly 4 candies in a straight line
- **Appearance**: Same color as matched candies with horizontal or vertical stripes
- **Stripe Direction**: Determined by swipe direction (horizontal swipe = horizontal stripes, vertical swipe = vertical stripes)
- **Effect**: Clears entire row (horizontal stripes) or column (vertical stripes) from candy's position
- **Points**: 120 for creation, row/column candies score individually

#### Wrapped Candy
- **Creation**: Match 5 candies in L-shape or T-shape
- **Appearance**: Same color with wrapped/bow appearance
- **Effect**: Explodes twice
  - First explosion: Clears 3×3 area centered on candy
  - Candy then drops into new position
  - Second explosion: Clears another 3×3 area
- **Points**: 200 for creation, 540 total for explosions

#### Color Bomb
- **Creation**: Match 5+ candies in a straight line
- **Appearance**: Chocolate ball with rainbow sprinkles (no specific color)
- **Activation**: Must be swapped with another candy (cannot be matched)
- **Effect**: Removes ALL candies of the swapped candy's color from entire board via lightning effects
- **Secondary Activation**: If destroyed by special candy effect (not swapped), targets most common color on board
- **Points**: 200 for creation, 20 per candy destroyed

#### Special Candy Combinations
When two special candies are swapped together, their effects multiply:

| Combination | Effect |
|-------------|--------|
| Striped + Striped | Clears one full row AND one full column (cross pattern) |
| Striped + Wrapped | Creates "giant candy" clearing 3 rows AND 3 columns |
| Striped + Color Bomb | All candies of striped's color become striped candies, then detonate sequentially |
| Wrapped + Wrapped | Massive double explosion: 6×5 rectangle, then two 5×5 squares |
| Wrapped + Color Bomb | All candies of wrapped's color become wrapped candies, then detonate |
| Color Bomb + Color Bomb | Complete board clear, removes one layer from all blockers |

---

### Level Objective Types

Each level has one primary objective type (or mixed). Level is won when all objectives are completed before moves run out.

#### Jelly Levels
- **Objective**: Clear all jelly from the board
- **Jelly Types**: 
  - Single jelly (translucent colored layer): 1 match to clear
  - Double jelly (white/opaque layer): 2 matches to clear (becomes single, then clears)
- **Mechanic**: Jelly only clears when the candy ON TOP of it is destroyed
- **Strategy Note**: Corner and edge jellies are hardest to reach
- **End Bonus**: Remaining moves spawn 3 Jelly Fish each

#### Ingredient Levels
- **Objective**: Navigate ingredients (cherries, hazelnuts) to exit points (green arrows at board edges)
- **Mechanic**: 
  - Ingredients fall like candies but CANNOT be matched
  - Ingredients only move when candies below them are cleared
  - Horizontal movement requires vertical matches adjacent to ingredient
- **Exit Points**: Typically at bottom of board, marked with green arrows
- **Strategy Note**: Vertical striped candies are most valuable

#### Order Levels
- **Objective**: Collect specific items within move limit
- **Possible Orders**:
  - Specific candy colors (e.g., "Collect 50 red candies")
  - Special candies (e.g., "Create 3 wrapped candies")
  - Specific combinations (e.g., "Make 2 Striped + Wrapped combos")
  - Blockers (e.g., "Clear 20 frosting blocks")
- **Multiple Orders**: Levels can require completing multiple order types
- **End Bonus**: Remaining moves become striped candies that auto-activate

#### Mixed Mode Levels
- **Objective**: Complete ALL objectives from multiple types
- **Example**: Clear all jelly AND drop 3 ingredients
- **Interaction**: If ingredient sits on jelly, destroying ingredient clears the jelly

---

### Blockers and Obstacles

Blockers create puzzle complexity by interfering with matches, movement, and special candy effects.

#### Frosting (Meringue)
- **Layers**: 1-5 layers, requiring that many hits to clear
- **Behavior**: Stationary, blocks candy from occupying its space
- **Clearing**: Adjacent matches OR special candy effects
- **Visual**: White/cream colored, layers visible as height/opacity

#### Chocolate
- **Behavior**: SPREADS if not cleared each turn
- **Spreading Rule**: If no chocolate cleared on player's turn, one chocolate piece consumes an adjacent candy
- **Clearing**: Adjacent matches OR special candy effects
- **Chocolate Fountain**: Regenerates chocolate every 2 moves when board is clear of chocolate
- **Visual**: Brown, glossy appearance

#### Licorice Swirl
- **Behavior**: Moves like candy but CANNOT be matched
- **Clearing**: Adjacent matches only
- **Special Property**: BLOCKS striped candy effects (stops the line blast)
- **Visual**: Black and red spiral

#### Marmalade (Honey)
- **Behavior**: Transparent coating over candies, prevents activation
- **Clearing**: One adjacent match removes the coating
- **Visual**: Golden/orange translucent layer

#### Licorice Lock
- **Behavior**: Immobilizes candy, prevents swapping or falling
- **Clearing**: Match the locked candy itself OR hit with special candy effect
- **Visual**: Black licorice cage around candy

#### Candy Bomb
- **Behavior**: Countdown decreases each move, level FAILS if any bomb reaches 0
- **Clearing**: Match like regular candy (has a color) OR hit with special effects
- **Visual**: Spherical bomb with number displayed, color-coded
- **Warning**: Visual pulse and sound when countdown ≤ 3

#### Sugar Chest
- **Behavior**: Contains candy that cannot be accessed until unlocked
- **Clearing**: ONLY by matching Sugar Keys (special element that spawns)
- **Immunity**: Even Color Bomb + Color Bomb cannot clear chests without keys
- **Visual**: Treasure chest with lock

---

### Boosters

#### Pre-Game Boosters (Selected before level starts)
- **Color Bomb Booster**: Places 1 Color Bomb on starting board
- **Striped & Wrapped Booster**: Places 1 striped and 1 wrapped candy (not adjacent)
- **Extra Moves (+5)**: Adds 5 moves to level's move limit

#### In-Game Boosters (Used during play, do NOT consume moves)
- **Lollipop Hammer**: Destroys any single candy or removes 1 layer from blocker. Triggers special candy effects if hitting them.
- **Free Switch**: Swap any two adjacent candies without requiring a match
- **Shuffle**: Rearranges all candies on board (like automatic shuffle)

---

### UI Layout

#### Game Screen Components

```
┌─────────────────────────────────────────┐
│  [≡]     LEVEL 42        [⚙️]          │  ← Header: Menu, Level #, Settings
├─────────────────────────────────────────┤
│                                         │
│   ┌─────────────────────────────────┐   │
│   │  🎯 Clear 20 Jellies    [15/20] │   │  ← Objective Panel
│   │  🍬 Collect 30 Red      [22/30] │   │
│   └─────────────────────────────────┘   │
│                                         │
│   ┌───┐                                 │
│   │25 │  MOVES          ⭐ 12,450      │  ← Moves Counter & Score
│   └───┘                                 │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │ ○ ○ ○ ○ ○ ○ ○ ○ ○ │             │   │
│   │ ○ ● ○ ○ ○ ○ ● ○ ○ │             │   │  ← Game Board (9×9 max)
│   │ ○ ○ ○ ○ ○ ○ ○ ○ ○ │             │   │
│   │ ○ ○ ● ○ ○ ○ ○ ○ ○ │             │   │
│   │ ○ ○ ○ ○ ○ ○ ○ ● ○ │             │   │
│   │ ○ ○ ○ ○ ● ○ ○ ○ ○ │             │   │
│   │ ○ ○ ○ ○ ○ ○ ○ ○ ○ │             │   │
│   │ ○ ○ ○ ○ ○ ○ ○ ○ ○ │             │   │
│   │ ○ ○ ○ ○ ○ ○ ○ ○ ○ │             │   │
│   └─────────────────────────────────┘   │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │  🔨    🔄    💫                  │   │  ← Booster Tray
│   │ Hammer Switch Shuffle           │   │
│   └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

#### Level Complete Screen
- Star rating (1-3 stars based on score thresholds)
- Final score with animated counter
- "Next Level" and "Replay" buttons
- Celebration particles and animations

#### Level Failed Screen
- "Out of Moves!" message
- Current progress toward objectives
- "Try Again" and "Main Menu" buttons
- Subdued animation (no celebration)

#### Main Menu / Level Select
- World map with episode progression
- Levels displayed as numbered nodes
- Locked levels grayed out
- Star count displayed on completed levels

---

### Game Feel and Feedback Systems

The "juiciness" of the game is critical to player satisfaction. Every action must feel impactful and rewarding.

#### Audio Feedback

**Match Sounds**
- Each match plays a "pop" sound
- CASCADE ESCALATION: Each subsequent match in a cascade plays the SAME sound but ONE SEMI-TONE HIGHER
- This creates ascending musical progressions during combos
- Maximum escalation: 12 semi-tones (one octave)

**Voice Lines** (trigger at cascade thresholds)
- 2-chain: "Sweet!"
- 3-chain: "Tasty!"
- 4-chain: "Divine!"
- 5+ chain: "Delicious!"
- Level complete: "Sugar Crush!"

**Special Candy Sounds**
- Striped activation: Whoosh/slice sound
- Wrapped activation: Explosion sound (plays twice)
- Color Bomb activation: Lightning crackle + magical chime
- Combination activation: Layered version of both sounds

**Negative Feedback**
- Invalid swap: Brief, soft error tone
- Level failed: Gentle descending piano scale (NOT harsh)

#### Visual Feedback

**Match Animations**
- Matched candies scale to 0 with ease-in curve (0.1s)
- Particle burst at each destroyed candy position
- Particles color-matched to candy

**Cascade Visual Escalation**
| Cascade Level | Visual Effect |
|---------------|---------------|
| 1 (basic match) | Small pop, minimal particles |
| 2 | Medium burst, "Sweet!" text popup |
| 3 | Large explosion, "Tasty!" text popup |
| 4 | Screen flash, heavy particles |
| 5+ | Full screen effects, "Delicious!" text, screen shake |

**Special Candy Visuals**
- Striped: Line blast travels across row/column with trail effect
- Wrapped: Expanding circular shockwave (twice)
- Color Bomb: Lightning bolts connecting to each targeted candy

**General Polish**
- All movement uses ease-in-out curves (never linear)
- Candies "squash and stretch" on interactions
- Slight bounce on candy landing after fall
- Idle animation: Subtle candy wobble/shimmer
- Selected candy: Glowing outline pulse

**Score Display**
- Points fly from destroyed candies toward score counter
- Score counter animates incrementally (not instant)
- Large score gains trigger counter "explosion" effect

---

## Technical Implementation

### Technology Stack
- React 18+ with functional components and hooks
- CSS Modules or Tailwind CSS for styling
- Howler.js or Web Audio API for sound
- Framer Motion or React Spring for animations
- TypeScript recommended for type safety

### Data Models

#### Game State
```typescript
interface GameState {
  board: (Candy | Blocker | null)[][];
  rows: number;
  cols: number;
  moves: number;
  maxMoves: number;
  score: number;
  objectives: Objective[];
  currentCascadeLevel: number;
  isProcessing: boolean;
  selectedCell: Position | null;
  gameStatus: 'playing' | 'won' | 'lost';
  levelConfig: LevelConfig;
}
```

#### Candy
```typescript
interface Candy {
  type: 'candy';
  id: string; // unique identifier for React keys
  color: CandyColor;
  special: SpecialType | null;
  stripeDirection?: 'horizontal' | 'vertical';
  position: Position;
  isMatched: boolean;
  isNew: boolean; // for spawn animation
}

type CandyColor = 'blue' | 'green' | 'orange' | 'purple' | 'red' | 'yellow';
type SpecialType = 'striped' | 'wrapped' | 'colorBomb';
```

#### Blocker
```typescript
interface Blocker {
  type: 'blocker';
  blockerType: BlockerType;
  layers: number; // for multi-layer blockers
  countdown?: number; // for candy bombs
  color?: CandyColor; // for candy bombs
  lockedCandy?: Candy; // for marmalade/locks
  position: Position;
}

type BlockerType = 'frosting' | 'chocolate' | 'licoriceSwirl' | 'marmalade' | 'licoriceLock' | 'candyBomb' | 'sugarChest';
```

#### Level Configuration
```typescript
interface LevelConfig {
  levelNumber: number;
  boardLayout: CellType[][]; // 'normal' | 'blocked' | 'exit' | 'jelly' | 'doubleJelly'
  initialBoard: (CandyColor | BlockerType | null)[][];
  objectives: ObjectiveConfig[];
  moves: number;
  starThresholds: [number, number, number]; // score needed for 1, 2, 3 stars
  availableColors: CandyColor[]; // typically 4-6 colors
  chocolateFountains?: Position[];
  spawners?: SpawnerConfig[]; // for ingredients, keys, etc.
}
```

#### Objective
```typescript
interface Objective {
  type: ObjectiveType;
  target: number;
  current: number;
  completed: boolean;
  itemType?: CandyColor | SpecialType | BlockerType;
}

type ObjectiveType = 'score' | 'jelly' | 'ingredient' | 'collectCandy' | 'collectSpecial' | 'clearBlocker';
```

#### Position
```typescript
interface Position {
  row: number;
  col: number;
}
```

### Core Algorithms

#### Match Detection Algorithm
```
function findMatches(board):
  matches = []
  
  // Horizontal scan
  for each row:
    streak = [first candy]
    for each candy in row (starting from second):
      if candy.color == streak[0].color:
        streak.push(candy)
      else:
        if streak.length >= 3:
          matches.push(streak)
        streak = [candy]
    if streak.length >= 3:
      matches.push(streak)
  
  // Vertical scan (same logic, iterate columns)
  
  // Merge overlapping matches for special candy detection
  return mergeOverlappingMatches(matches)
```

#### Special Candy Creation Logic
```
function determineSpecialCandy(matchedPositions, swipeDirection):
  shape = analyzeMatchShape(matchedPositions)
  
  if shape.isStraightLine:
    if matchedPositions.length == 4:
      return { type: 'striped', direction: swipeDirection }
    if matchedPositions.length >= 5:
      return { type: 'colorBomb' }
  
  if shape.isLShape or shape.isTShape:
    return { type: 'wrapped' }
  
  return null // regular match, no special
```

#### Cascade Loop
```
async function processCascade(board):
  cascadeLevel = 0
  
  while true:
    matches = findMatches(board)
    if matches.length == 0:
      break
    
    cascadeLevel++
    playMatchSound(cascadeLevel)
    
    // Process matches
    for each match in matches:
      specialCandy = determineSpecialCandy(match)
      destroyCandies(match)
      updateScore(match, cascadeLevel)
      updateObjectives(match)
      
      if specialCandy:
        createSpecialCandy(specialCandy, match.position)
    
    await animateDestruction()
    
    // Apply gravity
    dropCandies(board)
    await animateFalling()
    
    // Spawn new candies
    fillEmptySpaces(board)
    await animateSpawning()
  
  checkWinLoseConditions()
  returnControlToPlayer()
```

#### Weighted Random Candy Selection
```
function selectRandomCandy(board, availableColors):
  colorCounts = countColorsOnBoard(board)
  weights = {}
  
  for each color in availableColors:
    // Lower weight for colors that appear more frequently
    weights[color] = 1 / (colorCounts[color] + 1)
  
  return weightedRandomSelect(availableColors, weights)
```

### Component Architecture

```
App
├── GameProvider (Context for game state)
│   ├── MainMenu
│   │   └── LevelSelect
│   │       └── LevelNode (for each level)
│   │
│   └── GameScreen
│       ├── Header
│       │   ├── MenuButton
│       │   ├── LevelNumber
│       │   └── SettingsButton
│       │
│       ├── ObjectivePanel
│       │   └── ObjectiveItem (for each objective)
│       │
│       ├── StatsBar
│       │   ├── MovesCounter
│       │   └── ScoreDisplay
│       │
│       ├── GameBoard
│       │   ├── BoardCell (for each cell)
│       │   │   ├── Candy
│       │   │   ├── Blocker
│       │   │   └── Jelly (background layer)
│       │   └── EffectsLayer (particles, line blasts)
│       │
│       ├── BoosterTray
│       │   └── BoosterButton (for each booster)
│       │
│       └── Modals
│           ├── LevelCompleteModal
│           ├── LevelFailedModal
│           └── PauseMenu
│
└── AudioManager (sound effects and music)
```

### State Management

Use React Context + useReducer for game state:

```typescript
type GameAction =
  | { type: 'SELECT_CELL'; position: Position }
  | { type: 'SWAP_CANDIES'; from: Position; to: Position }
  | { type: 'PROCESS_MATCHES'; matches: Match[] }
  | { type: 'DROP_CANDIES' }
  | { type: 'SPAWN_CANDIES'; newCandies: Candy[] }
  | { type: 'UPDATE_SCORE'; points: number }
  | { type: 'UPDATE_OBJECTIVE'; objectiveIndex: number; progress: number }
  | { type: 'USE_BOOSTER'; boosterType: BoosterType; target?: Position }
  | { type: 'SET_PROCESSING'; isProcessing: boolean }
  | { type: 'CHECK_WIN_LOSE' }
  | { type: 'RESET_LEVEL' }
  | { type: 'LOAD_LEVEL'; levelConfig: LevelConfig };
```

### Animation Timing Constants
```typescript
const TIMING = {
  SWAP_DURATION: 150,        // ms
  INVALID_SWAP_DURATION: 250,
  MATCH_DESTROY_DURATION: 100,
  CANDY_FALL_DURATION: 200,  // per row
  SPAWN_DURATION: 150,
  SPECIAL_ACTIVATION: 300,
  CASCADE_DELAY: 500,        // delay between cascade iterations
  SCORE_FLY_DURATION: 400,
};
```

---

## Style Guide

### Color Palette

**Candy Colors**
- Blue: `#4A90D9` (primary), `#2E5A8C` (shadow)
- Green: `#5CB85C` (primary), `#3D7A3D` (shadow)
- Orange: `#F5A623` (primary), `#B8791A` (shadow)
- Purple: `#9B59B6` (primary), `#6C3483` (shadow)
- Red: `#E74C3C` (primary), `#A93226` (shadow)
- Yellow: `#F1C40F` (primary), `#B8960B` (shadow)

**UI Colors**
- Background: `#1A1A2E` (dark purple-blue)
- Panel Background: `#16213E` with 80% opacity
- Primary Accent: `#E94560` (pink-red)
- Secondary Accent: `#0F3460` (deep blue)
- Text Primary: `#FFFFFF`
- Text Secondary: `#A0A0A0`

### Typography
- Headers: Bold, rounded sans-serif (e.g., "Nunito", "Quicksand")
- Body: Regular weight, high readability
- Score/Numbers: Bold, slightly condensed for impact

### Visual Style
- Rounded corners on all elements (8-16px border-radius)
- Subtle drop shadows for depth
- Glossy/candy-like appearance on game elements
- Particle effects use soft, glowing circles
- Animations should feel "bouncy" and playful

---

## Testing Scenarios

### Match Detection Tests
1. Basic 3-match horizontal creates no special, clears 3 candies
2. Basic 3-match vertical creates no special, clears 3 candies
3. 4-match creates striped candy at last-moved position
4. 5-match straight creates color bomb
5. L-shape creates wrapped candy at intersection
6. T-shape creates wrapped candy at intersection
7. Two separate matches on same move both process correctly
8. Overlapping matches (cross pattern) create single special, not two

### Cascade Tests
1. Cascade scoring increases correctly (+120 per level)
2. Multiple cascades play ascending audio tones
3. Cascades stop when no more matches form
4. New candies spawn with correct weighted randomness

### Special Candy Tests
1. Striped candy clears correct row/column
2. Wrapped candy explodes twice with movement between
3. Color bomb removes all of target color
4. All 6 special-special combinations work correctly
5. Special candy destroyed by another special activates its effect

### Blocker Tests
1. Frosting requires correct number of hits per layer
2. Chocolate spreads when not cleared
3. Licorice swirl blocks striped candy line blast
4. Candy bomb countdown decreases each move
5. Candy bomb reaching 0 causes immediate level failure
6. Sugar chest only opens with key match

### Objective Tests
1. Jelly clears when candy on top is destroyed
2. Double jelly becomes single jelly, then clears
3. Ingredients move down when candy below clears
4. Ingredients reach exit and count toward objective
5. Order objectives track correct item collection
6. Mixed objectives all must complete to win

### Edge Cases
1. No valid moves triggers automatic shuffle
2. Shuffle preserves special candies and blockers
3. Post-shuffle always has at least one valid move
4. Swap during cascade is prevented (isProcessing = true)
5. Board with only blockers and no candies handles gracefully

---

## Accessibility Requirements

### Visual Accessibility
- Candies must be distinguishable by SHAPE, not just color (for colorblind players)
- Provide colorblind mode toggle with distinct patterns/symbols on each candy color
- Minimum contrast ratio 4.5:1 for all text
- Animation reduction option for motion-sensitive players
- Screen reader support for game state announcements

### Motor Accessibility
- Touch targets minimum 44×44 pixels
- Drag-to-swap alternative: tap source, tap destination
- No time-pressure mechanics (all levels are move-based, not timed)
- One-handed playable in portrait mode

### Audio Accessibility
- All audio feedback has visual equivalent
- Subtitles/captions for voice lines
- Volume controls for music, SFX, voice separately
- Game fully playable with sound muted

---

## Performance Goals

### Target Metrics
- 60 FPS during all animations
- Initial load under 3 seconds on 4G connection
- Memory usage under 100MB
- Touch response latency under 100ms

### Optimization Strategies
- Use React.memo for candy components (only re-render on actual changes)
- Implement object pooling for particles
- Lazy load audio files
- Preload next level assets during current level play
- Use CSS transforms for animations (GPU-accelerated)
- Batch state updates during cascade processing

---

## Extended Features (Optional Enhancements)

These features are not required for core functionality but enhance the experience:

### Progressive Difficulty
- Tutorial levels introduce one mechanic at a time
- First blocker appears after player masters basic matching
- Difficulty spikes followed by "breather" levels
- New mechanics debut in easier implementations before combining

### Level Editor (Development Tool)
- Grid-based placement of candies, blockers, objectives
- Test play functionality
- Export/import level configurations as JSON

### Statistics Tracking
- Total stars earned
- Levels completed
- Highest cascade achieved
- Most used boosters

### Social Features
- Share high scores
- Challenge friends on specific levels
- Weekly leaderboards

---

## File Structure

```
src/
├── components/
│   ├── game/
│   │   ├── GameBoard.tsx
│   │   ├── BoardCell.tsx
│   │   ├── Candy.tsx
│   │   ├── Blocker.tsx
│   │   ├── JellyLayer.tsx
│   │   └── EffectsLayer.tsx
│   ├── ui/
│   │   ├── Header.tsx
│   │   ├── ObjectivePanel.tsx
│   │   ├── StatsBar.tsx
│   │   ├── BoosterTray.tsx
│   │   └── Button.tsx
│   ├── modals/
│   │   ├── LevelCompleteModal.tsx
│   │   ├── LevelFailedModal.tsx
│   │   └── PauseMenu.tsx
│   └── menu/
│       ├── MainMenu.tsx
│       └── LevelSelect.tsx
├── context/
│   └── GameContext.tsx
├── hooks/
│   ├── useGameLogic.ts
│   ├── useMatchDetection.ts
│   ├── useCascade.ts
│   ├── useAnimation.ts
│   └── useAudio.ts
├── utils/
│   ├── matchDetection.ts
│   ├── specialCandyLogic.ts
│   ├── cascadeProcessor.ts
│   ├── boardUtils.ts
│   └── scoringSystem.ts
├── data/
│   └── levels/
│       ├── level1.json
│       ├── level2.json
│       └── ...
├── types/
│   └── index.ts
├── constants/
│   ├── timing.ts
│   ├── scoring.ts
│   └── colors.ts
├── assets/
│   ├── images/
│   ├── sounds/
│   └── fonts/
├── styles/
│   ├── global.css
│   └── variables.css
└── App.tsx
```

---

## Implementation Priority

### Phase 1: Core Engine (MVP)
1. Basic board rendering with 6 candy colors
2. Swap mechanics with validation
3. Match detection (3-match only)
4. Cascade system with gravity and spawning
5. Basic score tracking
6. Win condition (reach target score)

### Phase 2: Special Candies
1. Striped candy creation and activation
2. Wrapped candy creation and activation
3. Color bomb creation and activation
4. All special-special combinations
5. Proper animation for each special type

### Phase 3: Level Objectives
1. Jelly levels with single/double jelly
2. Ingredient levels with exit points
3. Order levels with collection tracking
4. Mixed mode objective handling
5. Star rating system

### Phase 4: Blockers
1. Frosting (multi-layer)
2. Chocolate (spreading)
3. Licorice swirl
4. Candy bomb (countdown)
5. Remaining blockers

### Phase 5: Polish
1. Full audio implementation with cascade escalation
2. Voice lines at cascade thresholds
3. Particle effects and screen shake
4. UI animations and transitions
5. Level select and progression

### Phase 6: Content
1. Design 50+ levels with progressive difficulty
2. Tutorial sequence
3. Booster system
4. Settings and accessibility options
