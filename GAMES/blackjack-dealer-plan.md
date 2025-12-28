# Blackjack with Live Animated Dealer

A React-based Blackjack game featuring an HD 2D animated dealer character with text-to-speech lip-sync and conversational AI capabilities.

---

## Overview

The game combines classic Blackjack mechanics with an interactive animated dealer who responds to gameplay events, engages in conversation, and brings personality to the table through a customisable persona system.

---

## Core Architecture

### Component Structure

```
<App>
├── <GameProvider>          // State management context
├── <DealerCanvas>          // 2D character rendering
│   ├── <SpriteAnimator>    // Frame-based animation controller
│   └── <LipSyncController> // TTS mouth shape mapping
├── <GameTable>
│   ├── <DealerHand>
│   ├── <PlayerHand>
│   ├── <ChipStack>
│   └── <ActionButtons>     // Hit, Stand, Double, Split
├── <ChatInterface>
│   ├── <MessageHistory>
│   └── <ChatInput>
└── <PersonaEditor>         // Markdown system prompt config
```

---

## Dealer Character System

### Visual Specifications

| Attribute | Value |
|-----------|-------|
| Resolution | 512×768 HD sprite sheets |
| Frame Rate | 8–12 fps (stylised low-rate aesthetic) |
| Art Style | 2D illustrated, semi-realistic or stylised |
| Layers | Base body, face, mouth shapes, accessories |

### Animation States

**Idle Animations**
- `idle_neutral` — subtle breathing, occasional blinks
- `idle_waiting` — finger tap, slight sway
- `idle_shuffle` — card shuffling loop

**Game Event Animations**
- `deal_card` — arm sweep dealing motion
- `reveal_card` — dramatic flip gesture
- `collect_chips` — sliding chips toward house
- `push_chips` — sliding chips toward player

**Performance Reactions**
- `player_win` — smile, nod, light applause
- `player_bust` — sympathetic shrug
- `player_blackjack` — impressed reaction, eyebrow raise
- `dealer_bust` — disappointed head shake
- `big_win` — celebration, thrown hands

**Communication Animations**
- `talking` — looped with lip-sync overlay
- `emote_laugh` — triggered by humour in chat
- `emote_think` — pondering gesture for complex questions
- `emote_greeting` — wave, welcoming gesture

---

## Lip-Sync TTS Integration

### Pipeline

```
User Input / Game Event
        ↓
   AI Response Text
        ↓
   TTS Engine (Web Speech API or external)
        ↓
   Phoneme/Viseme Extraction
        ↓
   Mouth Shape Mapping → SpriteAnimator
        ↓
   Synchronised Audio + Animation Playback
```

### Viseme Set (Simplified)

| Viseme | Phonemes | Mouth Shape |
|--------|----------|-------------|
| A | a, ah, ay | Open wide |
| E | e, ee | Wide smile |
| I | i, ih | Narrow open |
| O | o, oh | Round |
| U | oo, w | Pursed |
| M | m, b, p | Closed |
| F | f, v | Teeth on lip |
| REST | silence | Neutral closed |

---

## Chat & Conversation System

### Features

- Persistent chat panel alongside game table
- Context-aware responses (dealer knows game state)
- Natural conversation flow during gameplay
- Dealer initiates comments on notable events

### Context Injection

The AI receives structured game state with each message:

```json
{
  "playerHand": ["K♠", "7♥"],
  "playerTotal": 17,
  "dealerShowing": "5♦",
  "currentBet": 50,
  "playerBalance": 450,
  "roundOutcome": null,
  "sessionStats": { "wins": 3, "losses": 2 }
}
```

---

## Persona Customisation

### Markdown System Prompt Format

Users can define dealer personality via simple markdown:

```markdown
# Dealer Persona

## Name
Vic "Lucky" Malone

## Background
Former Vegas pit boss turned friendly neighbourhood dealer.
30 years at the tables. Seen it all.

## Personality Traits
- Warm and encouraging
- Dry wit, occasional dad jokes
- Offers strategy tips when asked
- Never condescending about losses

## Speech Style
- Uses classic casino lingo
- Calls player "friend" or "pal"
- Short, punchy sentences
- Occasional "back in my day" stories

## Boundaries
- No real gambling advice
- Keeps things light and fun
- Redirects if conversation gets too personal
```

### Preset Personas

| Persona | Style |
|---------|-------|
| Classic Dealer | Professional, minimal chatter |
| Friendly Host | Warm, encouraging, chatty |
| Noir Character | Mysterious, cryptic, atmospheric |
| Comedy Dealer | Jokes, puns, playful banter |
| Tutorial Guide | Educational, explains odds and strategy |

---

## Animation Trigger System

### Event-to-Animation Mapping

```javascript
const animationTriggers = {
  // Game Events
  'DEAL_START':       'deal_card',
  'PLAYER_HIT':       'deal_card',
  'PLAYER_STAND':     'idle_waiting',
  'DEALER_REVEAL':    'reveal_card',
  
  // Outcomes
  'PLAYER_BLACKJACK': 'player_blackjack',
  'PLAYER_WIN':       'player_win',
  'PLAYER_BUST':      'player_bust',
  'DEALER_BUST':      'dealer_bust',
  'PUSH':             'idle_neutral',
  
  // Chat
  'AI_SPEAKING':      'talking',
  'AI_IDLE':          'idle_neutral',
  
  // Emotes (sentiment analysis on AI response)
  'EMOTE_HAPPY':      'emote_laugh',
  'EMOTE_THINKING':   'emote_think'
};
```

### Priority Queue

Animations are queued with priority levels to handle overlapping triggers:

1. **Critical** — Game outcomes, dealing
2. **Standard** — Chat responses, reactions
3. **Background** — Idle loops, ambient motion

---

## Technical Stack

| Layer | Technology |
|-------|------------|
| UI Framework | React 18+ with hooks |
| State Management | Context API or Zustand |
| Animation | Canvas API or PixiJS |
| TTS | Web Speech API / ElevenLabs |
| AI Chat | Anthropic API (Claude) |
| Styling | Tailwind CSS |
| Audio | Howler.js |

---

## Future Considerations

- Multiple dealer characters to choose from
- Unlockable cosmetics and table themes
- Multiplayer spectator mode
- Voice input for hands-free play
- Mobile-optimised touch controls
- Statistics and hand history tracking

---

## Summary

This architecture delivers an immersive single-player Blackjack experience where the animated dealer feels like a genuine presence at the table—reacting to wins and losses, engaging in conversation, and bringing customisable personality to every hand dealt.
