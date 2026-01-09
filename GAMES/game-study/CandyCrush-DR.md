# Candy Crush Saga: Technical Game Design Document

**A complete match-3 game system built on cascading feedback loops, layered blockers, and meticulously designed "juiciness" creates one of gaming's most polished puzzle experiences.** This document breaks down every mechanical system that makes the game work—from core matching rules to the precise semi-tone audio escalation that makes combos feel rewarding. Each system interconnects: special candies counter specific blockers, level objectives dictate which special candies become valuable, and board configurations create strategic puzzles from simple rules.

---

## Core matching engine and board physics

The fundamental match-3 system operates on a **9×9 maximum grid** (though boards can be any shape within this boundary) with **6 candy colors**: blue, green, orange, purple, red, and yellow. The minimum valid match requires **3 identical candies in a horizontal or vertical line**—diagonal matches are never valid.

**Swap validation** follows strict rules. Players can only swap orthogonally adjacent candies, and a swap is only permitted if it creates at least one match. The adjacency check uses a simple Manhattan distance formula: `|tile1.x - tile2.x| + |tile1.y - tile2.y| == 1`. Invalid swaps trigger a "ping-pong" animation (approximately **0.25 seconds**) returning candies to their original positions.

**Match detection** runs after every swap, scanning the board in horizontal rows and vertical columns, counting consecutive same-colored candies. Any sequence of 3+ gets added to a match collection for batch removal. The detection algorithm must also identify match *shapes* for special candy creation—an L-shape versus a straight line produces different results.

When no valid moves exist anywhere on the board, the game automatically **shuffles** the candy positions. Critically, shuffles preserve the existing color distribution—they rearrange positions without adding or removing any colors. Every shuffle is guaranteed to produce at least one possible match. Objects immune to shuffling include blockers, special candies, licorice swirls, and ingredients.

### Cascading chain reactions

The cascade loop represents the game's most important mechanical system:

1. **Match detected** → Tiles flagged as "matched"
2. **Remove phase** → Matched tiles deleted, creating empty spaces
3. **Gravity phase** → Candies above drop down column-by-column
4. **Spawn phase** → New random candies enter from top
5. **Re-check phase** → Full board scanned for new matches
6. **Loop or exit** → If matches found, return to step 1; otherwise, return control to player

New candy spawning uses **weighted randomness**, not pure random selection. The weighting adjusts based on current board state to prevent clustering of same-colored candies and implements subtle "rubber-banding" that slightly helps struggling players.

**Animation timing** for the full cascade sequence typically breaks down as: match particles spawn at **0.0s**, tiles scale down with ease-in over **0.1s**, falling begins at **0.2s**, new tiles spawn at **0.3s**, and next match check occurs at **0.5s**. These timings must synchronize precisely—match detection on still-animating tiles creates bugs.

### Scoring cascades

The cascade scoring system rewards chain reactions quadratically. A base 3-candy match awards **60 points**, but cascade bonuses add **120 points** for the first cascade, **240** for the second, **360** for the third—increasing by 120 per level. This creates the powerful incentive to set up cascades rather than match immediately.

---

## Special candy creation and combination effects

Special candies form the strategic core of Candy Crush. Each requires a specific match pattern to create, and combining two special candies produces dramatically amplified effects.

### Creation patterns

| Special Candy | Pattern | Placement Rule |
|--------------|---------|----------------|
| **Striped Candy** | 4 candies in a straight line | Created at position of last-moved candy |
| **Wrapped Candy** | 5 candies in L or T shape | Appears at intersection point |
| **Color Bomb** | 5+ candies in a straight line | Created at position of last-moved candy |
| **Jelly Fish** | 4 candies in 2×2 square | Added in 2025 update |

**Striped candy direction** is determined by swipe direction: horizontal swipe creates horizontal stripes (clears row), vertical swipe creates vertical stripes (clears column). When activated, the stripe clears its entire row or column from the candy's position.

**Wrapped candies** explode twice—first clearing a **3×3 area** around the initial position, then dropping into a new position and exploding again. This double-explosion makes them exceptionally powerful against layered blockers.

**Color bombs** appear as chocolate balls with rainbow sprinkles, colorless until activated. When swapped with any candy, they remove all candies of that color from the entire board via lightning effects. If destroyed by another special candy's effect (rather than swapped), they automatically target the most prevalent color on the board.

### The combination matrix

When two special candies are swapped together, their effects combine multiplicatively:

**Striped + Striped** creates a cross pattern, clearing one full row AND one full column from the swap position. Individual stripe directions are ignored.

**Striped + Wrapped** produces a "giant candy" effect clearing **3 rows AND 3 columns**—effectively a massive plus sign. This is considered one of the most powerful combinations, capable of clearing most of the board.

**Striped + Color Bomb** transforms all candies of the striped candy's color into striped candies, then detonates them all sequentially. Stripes alternate between horizontal and vertical in reading order (left-to-right, top-to-bottom), creating dramatic board-clearing cascades.

**Wrapped + Wrapped** triggers a massive double explosion clearing approximately a **6×5 rectangle**, followed by two 5×5 square explosions—substantial but less versatile than Striped + Wrapped.

**Wrapped + Color Bomb** transforms all candies of the wrapped candy's color into wrapped candies, then detonates every one. The chain reaction of 3×3 explosions across the board devastates almost any configuration.

**Color Bomb + Color Bomb** is the rarest and most powerful combination: a complete board clear. Every candy is removed regardless of color, and one layer is removed from every blocker. This combination effectively wins most levels instantly.

---

## Level objective types

Candy Crush uses **5 active level types** that fundamentally change player strategy and which mechanics become valuable.

### Jelly levels dominate the difficulty curve

Appearing in **45% of all levels** (including mixed-mode), jelly levels require clearing translucent colored layers behind candies. Single jelly requires one match on that square; **double jelly** (white/opaque) requires two. Jelly can *only* be removed by destroying the candy sitting on top of it—special candy effects hitting jelly squares directly won't clear them unless they destroy a candy there.

Strategic implications: corners and edges become highest priority (hardest to reach), Jelly Fish boosters gain enormous value, and wrapped candies outperform striped candies due to their area effect. When completed, remaining moves each spawn **3 jelly fish** that target random squares.

### Ingredient levels require path planning

Players must navigate cherries and hazelnuts to designated exit points (green arrows, typically at board bottom). Ingredients fall like candies but **cannot be matched**—they only move when candies below them clear. Horizontal movement requires vertical matches adjacent to the ingredient.

Strategic implications: vertical striped candies become critical, players must plan paths before ingredients get stuck in "dead zones," and pre-game Coconut Wheel boosters (exclusive to ingredient levels) provide significant advantage.

### Order levels test collection efficiency

These levels require collecting specific items: regular candies of certain colors, special candies, specific combinations (like striped + wrapped), or clearing particular blockers. Orders can combine multiple requirements.

Strategic implications: orders requiring rare combinations (color bomb + color bomb) become heavily luck-dependent, Lucky Candy boosters (exclusive to order levels) can spawn any needed item, and players must prioritize harder-to-create orders first.

### Rainbow Rapids adds path-clearing mechanics

Introduced at level 7116, these require clearing blockers obstructing a rainbow stream flowing from a "faucet" to a "mold." Not every blocker on the board needs clearing—only those in the rainbow's predetermined path. Strategic focus narrows to path-blocking obstacles exclusively.

### Mixed mode combines objectives

**38% of all levels** combine two or more objectives from different types—the most common category. Players must complete ALL objectives, creating multiplicative complexity. Key interaction: if an ingredient sits on jelly, destroying the ingredient with a special candy clears the jelly too.

---

## Blockers and obstacles encyclopedia

Blockers create puzzle complexity by interfering with matches, movement, and special candy effects. Modern levels commonly combine **5-10 different blocker types**.

### Layered blockers require multiple hits

**Frosting/Meringue** (introduced Level 2) comes in 1-5 layers, requiring that many adjacent matches or special candy hits to clear. It's stationary and blocks candy from occupying its space. A 5-layer frosting over double jelly requires **7 total hits** to fully clear.

**Cake Bombs** occupy 2×2 tiles with **8 slices total** (2 hits per tile). When fully cleared, they trigger a powerful full-board explosion—making them beneficial when destroyable but problematic when blocking critical paths.

**Liquorice Shell** (formerly Popcorn) requires **3 special candy hits**—regular adjacent matches don't work. When finally cleared, it transforms into a Color Bomb, creating significant strategic value.

### Spreading blockers create urgency

**Chocolate** spreads aggressively: if no chocolate is cleared on a player's turn, one chocolate piece consumes an adjacent candy. This creates constant pressure to manage chocolate before pursuing objectives. Chocolate fountains regenerate chocolate every 2 moves when the board is clear, making complete elimination impossible.

**Dark Chocolate** (Level 1003+) is more aggressive—it gains layers OR spreads if not cleared within a move, with variants from 1-3 layers. Dark chocolate fountains spawn pre-layered dark chocolate, dramatically increasing threat level.

### Encasing blockers protect contents

**Marmalade/Honey** creates transparent coatings over candies, preventing their activation until cleared with one adjacent match. Often used to protect special candies (preventing premature activation) or to create puzzles requiring specific sequences.

**Licorice Locks** immobilize candies, preventing swapping or falling. The lock must be destroyed by matching the locked candy itself or hitting it with special candy effects. Locked candy bombs become especially dangerous since they can't be repositioned.

**Sugar Chests** contain candies that cannot be accessed until opened by matching Sugar Keys—the ONLY method that works. Even Color Bomb + Color Bomb cannot clear sugar chests, making key management critical on chest-heavy levels.

### Mobile blockers and barriers

**Licorice Swirls** move around the board like candies but cannot be matched—only adjacent matches clear them. Critically, they **block striped candy effects**, stopping the line blast at their position. This makes wrapped candies more reliable for levels heavy with swirls.

**Candy Cane Fences** are permanent, indestructible barriers positioned between tiles (not on them). They prevent swapping, falling, matching, and chocolate spreading across their boundary—but striped candy effects pass through.

**Liquorice Fences** are the deadly variant: permanent barriers that block ALL special candy effects except Color Bomb + Color Bomb. Blockers trapped inside liquorice fences become nearly impossible to clear.

### Pressure blockers create fail conditions

**Candy Bombs** display countdown numbers (1-99) that decrease each move. If any bomb reaches zero, the level **immediately fails**—no recovery possible. Candy bombs have colors and must be matched like regular candies or hit with special effects. One beneficial quirk: spreading chocolate that engulfs a bomb destroys it safely.

**Magic Mixers** periodically spawn blockers (chocolate, frosting, candy bombs) from their position. They require **5 adjacent matches** to destroy—special candy effects cannot damage them. Destroying a mixer triggers a **5×5 explosion**. On Order levels, mixers can become beneficial by spawning needed blockers.

---

## Booster and power-up systems

Boosters divide into pre-game (modifying starting board state) and in-game (activated during play) categories. Critically, **no in-game booster consumes a move**.

### Pre-game boosters shape starting conditions

**Color Bomb Booster** places one Color Bomb at a random valid position on the starting board—available on all level types.

**Jelly Fish Booster** (jelly levels only) adds fish to the spawning pool AND places 3 fish on the starting board. Fish continue spawning during play, and when matched, 3 fish swim to random jellied squares.

**Coconut Wheel Booster** (ingredient levels only) adds wheels to the spawning pool and places one on the starting board. When swapped, the wheel rolls in the swipe direction, converting **3 candies in its path to striped candies** that immediately activate.

**Striped & Wrapped Booster** places one of each at random positions—not adjacent, so setup is required to combine them.

**Lucky Candy Booster** (order levels only) adds Lucky Candies to spawning. When matched, they transform into whatever item is needed for current order objectives.

### In-game boosters provide tactical options

**Lollipop Hammer** (first booster, Level 7) destroys any single candy or removes one layer from blockers. It clears jelly underneath, defuses candy bombs, and triggers special candy effects if hitting them. Cannot target: chocolate fountains, sugar chests, ingredients, or empty tiles.

**Free Switch** swaps any two adjacent candies without requiring a match and without consuming a move. This enables strategic repositioning—especially valuable for setting up special candy combinations.

**Striped Brush** converts any regular candy into a striped candy, with player choice of stripe direction. The converted candy doesn't activate until matched.

**Bomb Cooler** adds +5 to ALL visible candy bombs' countdowns simultaneously—only appears when bombs are present.

**Party Popper** (most powerful) clears the entire board and spawns 4 special candies that immediately detonate, creating massive chain reactions.

### Sugar Crush converts remaining resources

Upon completing all objectives, Sugar Crush activates automatically. Each remaining move converts to bonus effects:

- **Jelly levels**: Each move spawns 3 jelly fish targeting random squares
- **Other levels**: Remaining moves become striped candies that auto-activate (approximately 4 stripes per 5 moves)
- **Base scoring**: 6,000 points per remaining move plus cascade bonuses

**Super Sugar Crush** triggers when completing objectives on the exact last move (0 remaining)—the entire board explodes like a Cake Bomb, with remaining special candies scoring bonus points (Color Bombs worth 5,000 each, Wrapped worth 1,000).

---

## Level design philosophy

King's level designers follow a structured approach combining intentional patterns, difficulty variance, and progressive complexity introduction.

### The three-act structure shapes every level

**Act 1 (Introduction)**: Player has space to understand progression with light challenge and room to experiment. Often uses "cramped corner" starts that open up or Sugar Chest locks that reveal playable area.

**Act 2 (Core Challenge)**: Main gimmick introduces with difficulty ramping significantly. Bulk of challenge placed here, often including pressure elements (spreading chocolate, countdown bombs) requiring multi-tasking.

**Act 3 (Finale)**: Challenge may peak but reward is imminent. Visual spectacle moments (chain reactions, massive explosions) as player works toward final goal. **Key insight**: designers often design the finale first, then work backwards.

### Symmetry signals intentionality

Research by modl.ai found human-designed levels distinguishable from AI-generated ones primarily through **intentional symmetry patterns**—both local (confined to sections) and global (spanning entire levels). This creates the sense of designed puzzles rather than random configurations.

### Difficulty curves embrace variance

Rather than linear progression, Candy Crush uses "schizophrenic difficulty"—hard levels frequently followed by easy "breather" levels. This maintains player confidence while creating challenge. New mechanics debut with easier implementations, spike in difficulty, then return to easier levels. Approximately **33% of each 15-level episode** targets high difficulty.

**Hard level characteristics**: limited moves relative to objectives, multiple layered blockers (5+ layer frosting), spreading obstacles, complex ingredient paths, rare combination requirements, bombs with tight countdowns, restricted board space.

**Easy level characteristics**: generous move allocation, open boards with good candy flow, single-objective focus, fewer blocker types, 4-5 candy colors only, natural cascade-to-special-candy flow.

### Progressive mechanic introduction

New elements are **isolated first**—players learn ONE thing. The Sour Skull tutorial exemplifies this: Tutorial Level 1 introduces core mechanic in isolation (one skull, one pedestal). Tutorial Level 2 adds complexity (more pedestals, special candy damage). Tutorial Level 3 integrates with existing mechanics (licorice blocks special effects).

This "complexity layering" ensures players master basics before unlocking advanced features, with early uses forgiving through extra move allocation.

---

## Game feel and feedback systems

Candy Crush's "juiciness"—defined as "giving players far more output than their simple inputs deserve"—creates one of gaming's most satisfying feedback loops.

### Audio design reinforces positivity

The core audio philosophy: **almost no negative reinforcement**. Invalid moves receive only a brief error sound quickly teaching rules. Failed levels play a gentle downward piano scale rather than harsh punishment sounds.

Match sounds use a brilliant escalation technique: each subsequent match in a cascade plays the **same sound but one semi-tone higher**. This creates ascending musical progressions during combos, producing a "feeling of elevation, building jubilance."

The iconic voice lines ("Sweet!" → "Tasty!" → "Divine!" → "Delicious!" → "Sugar Crush!") in a deep, sultry American tone trigger at cascade thresholds. These voice lines create emotional peaks and memorable catchphrases—players actively try to hear "Divine!" and "Delicious!" as achievement markers.

### Visual feedback layers with audio

Every visual event has corresponding audio. Screen shake accompanies major explosions, particle intensity scales with combo size, and white flashes punctuate the biggest chain reactions.

**Cascade visual escalation**:
- Basic match: small pop animation
- 2-chain: medium burst with "Sweet!" text
- 3-chain: large explosion with "Tasty!" text
- 4+ chain: screen flash with particles
- Massive cascade: full screen effects with "Delicious!"

**Animation timing** uses acceleration rather than linear movement—candies fall with physics, bounce slightly on landing, and employ "squash and stretch" principles on interactions. All transitions use ease-in/ease-out curves rather than instant changes.

### Celebration moments amplify achievement

Sugar Crush transforms level completion into a spectacle: remaining moves convert to special candies that auto-activate in sequence, fireworks and particles explode across the screen, the score counter rapidly tallies up, and stars fill with animated glows. Even small wins (completing any match) receive positive feedback through particles and sounds.

The **dopamine loop** operates through immediate rewards (sound/visual on every action), variable ratio reinforcement (cascades unpredictable but frequent), constant visible progress, and "near miss" mechanics keeping players engaged.

### Polish principles for implementation

1. **Particles everywhere**: explosions, sparkles, trails on every interaction
2. **Sound layering**: multiple audio elements for single actions
3. **Screen response**: shake, flash, zoom scaled to impact
4. **Escalation**: bigger actions = proportionally bigger feedback
5. **Immediacy**: zero delay between input and feedback
6. **Celebrate everything**: even small wins deserve acknowledgment
7. **Positive over punishment**: minimize negative feedback entirely
8. **Pitch progression**: ascending tones for cascading events
9. **Voice creates emotion**: catchphrases build player connection

---

## Conclusion: systems that interconnect

Candy Crush's mechanical brilliance lies in how every system reinforces others. Special candy creation patterns make certain match shapes strategically valuable. Level objectives determine which special candies become priorities—jelly fish on jelly levels, vertical stripes on ingredient levels, lucky candies on order levels. Blockers counter specific special candy types—licorice swirls block stripes but not wraps. Combinations create multiplicative power encouraging special candy hoarding and setup.

The feedback systems transform these interconnected mechanics into satisfying experiences. Cascades feel rewarding through escalating audio and visual feedback. Special candy combinations produce spectacular effects worthy of the setup required. Even failure receives gentle treatment, encouraging retry rather than frustration.

For developers implementing similar systems, the critical insight is **layered feedback proportional to action size**. Simple matches get simple sounds; five-match combos into cascades into special candy combinations get escalating celebrations. The game constantly communicates "that was good" in proportion to how good it actually was—training players toward optimal play through pure positive reinforcement.