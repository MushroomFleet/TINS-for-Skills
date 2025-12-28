# METAL SLUG - React JSX Side-Scroller Implementation Plan

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Technical Architecture](#2-technical-architecture)
3. [Game State Management](#3-game-state-management)
4. [Core Components](#4-core-components)
5. [Player System](#5-player-system)
6. [Enemy System](#6-enemy-system)
7. [Collision System](#7-collision-system)
8. [Level System](#8-level-system)
9. [Level Editor](#9-level-editor)
10. [Menu System](#10-menu-system)
11. [Audio System](#11-audio-system)
12. [Implementation Phases](#12-implementation-phases)

---

## 1. Project Overview

### Game Concept
A side-scrolling run-and-gun game inspired by Metal Slug. Players move left-to-right, shooting enemies and avoiding bullets to complete levels.

### Key Features
- Arcade-style main menu with options
- Level editor (debug mode)
- Side-scrolling gameplay
- Player with walk, jump, roll, shoot, kick, and capture mechanics
- Enemy "goons" with walk, punch, shoot, and death mechanics
- SVG placeholder sprites (3-frame animation system)
- Collision detection system

### Technical Stack
- React 18+ with Hooks
- JSX for component structure
- SVG for placeholder graphics
- CSS for styling
- RequestAnimationFrame for game loop

---

## 2. Technical Architecture

### File Structure
```
/metal-slug-game
├── components/
│   ├── Game.jsx                 # Main game container
│   ├── Menu/
│   │   ├── MainMenu.jsx         # Title screen
│   │   ├── OptionsMenu.jsx      # Settings
│   │   └── PauseMenu.jsx        # In-game pause
│   ├── Gameplay/
│   │   ├── GameCanvas.jsx       # Main gameplay area
│   │   ├── Player.jsx           # Player component
│   │   ├── Enemy.jsx            # Enemy component
│   │   ├── Bullet.jsx           # Projectile component
│   │   ├── Platform.jsx         # Platform/ground
│   │   └── Background.jsx       # Scrolling background
│   ├── LevelEditor/
│   │   ├── Editor.jsx           # Level editor main
│   │   ├── Toolbar.jsx          # Placement tools
│   │   ├── PropertyPanel.jsx    # Object properties
│   │   └── LevelPreview.jsx     # Test play
│   └── UI/
│       ├── HUD.jsx              # Score, health, ammo
│       ├── DebugOverlay.jsx     # Debug info
│       └── GameOver.jsx         # End screen
├── hooks/
│   ├── useGameLoop.js           # RAF game loop
│   ├── useInput.js              # Keyboard handling
│   ├── useCollision.js          # Collision detection
│   └── useCamera.js             # Viewport scrolling
├── systems/
│   ├── physics.js               # Gravity, velocity
│   ├── collision.js             # AABB collision
│   ├── animation.js             # Sprite animation
│   └── levelLoader.js           # Level data parsing
├── data/
│   ├── levels/                  # Level JSON files
│   ├── sprites/                 # SVG/PNG sprites
│   └── constants.js             # Game constants
└── utils/
    ├── vector.js                # Vector math
    └── helpers.js               # Utility functions
```

### Core Constants
```javascript
// constants.js
export const GAME_CONFIG = {
  // Display
  SCREEN_WIDTH: 800,
  SCREEN_HEIGHT: 450,
  TILE_SIZE: 32,
  
  // Physics
  GRAVITY: 0.6,
  MAX_FALL_SPEED: 12,
  FRICTION: 0.85,
  
  // Player
  PLAYER_SPEED: 4,
  PLAYER_JUMP_FORCE: -12,
  PLAYER_ROLL_SPEED: 8,
  PLAYER_ROLL_DURATION: 400, // ms
  
  // Enemy
  ENEMY_SPEED: 1.5,
  ENEMY_AGGRO_RANGE: 300,
  ENEMY_ATTACK_RANGE: 40,
  ENEMY_SHOOT_RANGE: 250,
  
  // Combat
  BULLET_SPEED: 10,
  PLAYER_FIRE_RATE: 150, // ms between shots
  ENEMY_FIRE_RATE: 1500,
  KICK_RANGE: 45,
  KICK_DAMAGE: 100,
  
  // Animation
  FRAME_DURATION: 150, // ms per frame
  SPRITE_FRAMES: 3,
};

export const GAME_STATES = {
  MENU: 'menu',
  PLAYING: 'playing',
  PAUSED: 'paused',
  LEVEL_EDITOR: 'level_editor',
  GAME_OVER: 'game_over',
  LEVEL_COMPLETE: 'level_complete',
};

export const PLAYER_STATES = {
  IDLE: 'idle',
  WALKING: 'walking',
  JUMPING: 'jumping',
  FALLING: 'falling',
  ROLLING: 'rolling',
  SHOOTING: 'shooting',
  KICKING: 'kicking',
  CAPTURED: 'captured', // "hands up" state
};

export const ENEMY_STATES = {
  IDLE: 'idle',
  PATROLLING: 'patrolling',
  CHASING: 'chasing',
  ATTACKING: 'attacking',
  SHOOTING: 'shooting',
  HIT: 'hit',
  DYING: 'dying',
  DEAD: 'dead',
};

export const DIRECTIONS = {
  LEFT: -1,
  RIGHT: 1,
  UP: -1,
  DOWN: 1,
};
```

---

## 3. Game State Management

### Main Game State
```javascript
// Initial game state structure
const initialGameState = {
  // Core state
  gameState: GAME_STATES.MENU,
  currentLevel: 1,
  score: 0,
  
  // Camera
  camera: {
    x: 0,
    y: 0,
    shakeIntensity: 0,
  },
  
  // Player
  player: {
    x: 100,
    y: 300,
    vx: 0,
    vy: 0,
    width: 32,
    height: 48,
    direction: DIRECTIONS.RIGHT,
    state: PLAYER_STATES.IDLE,
    health: 100,
    ammo: Infinity, // pistol
    specialAmmo: 0,
    weapon: 'pistol',
    isGrounded: false,
    animationFrame: 0,
    invulnerable: false,
    invulnerableTimer: 0,
  },
  
  // Entities
  enemies: [],
  bullets: [],
  pickups: [],
  particles: [],
  
  // Level
  level: {
    width: 3200,
    height: 450,
    platforms: [],
    spawnPoints: [],
    endZone: { x: 3100, y: 0, width: 100, height: 450 },
  },
  
  // Debug
  debug: {
    enabled: false,
    showHitboxes: false,
    showFPS: true,
    godMode: false,
  },
  
  // Options
  options: {
    musicVolume: 0.7,
    sfxVolume: 0.8,
    difficulty: 'normal',
  },
};
```

### State Reducer Pattern
```javascript
// gameReducer.js
export function gameReducer(state, action) {
  switch (action.type) {
    case 'SET_GAME_STATE':
      return { ...state, gameState: action.payload };
      
    case 'UPDATE_PLAYER':
      return { ...state, player: { ...state.player, ...action.payload } };
      
    case 'ADD_ENEMY':
      return { ...state, enemies: [...state.enemies, action.payload] };
      
    case 'UPDATE_ENEMY':
      return {
        ...state,
        enemies: state.enemies.map((e, i) =>
          i === action.index ? { ...e, ...action.payload } : e
        ),
      };
      
    case 'REMOVE_ENEMY':
      return {
        ...state,
        enemies: state.enemies.filter((_, i) => i !== action.index),
      };
      
    case 'ADD_BULLET':
      return { ...state, bullets: [...state.bullets, action.payload] };
      
    case 'UPDATE_BULLETS':
      return { ...state, bullets: action.payload };
      
    case 'UPDATE_CAMERA':
      return { ...state, camera: { ...state.camera, ...action.payload } };
      
    case 'ADD_SCORE':
      return { ...state, score: state.score + action.payload };
      
    case 'LOAD_LEVEL':
      return { ...state, level: action.payload, enemies: action.enemies || [] };
      
    case 'TOGGLE_DEBUG':
      return { ...state, debug: { ...state.debug, enabled: !state.debug.enabled } };
      
    case 'RESET_GAME':
      return { ...initialGameState, options: state.options };
      
    default:
      return state;
  }
}
```

---

## 4. Core Components

### Main Game Component
```jsx
// Game.jsx
import React, { useReducer, useCallback, useEffect } from 'react';
import { gameReducer, initialGameState } from './gameReducer';
import { GAME_STATES } from './constants';
import MainMenu from './Menu/MainMenu';
import GameCanvas from './Gameplay/GameCanvas';
import LevelEditor from './LevelEditor/Editor';
import PauseMenu from './Menu/PauseMenu';
import GameOver from './UI/GameOver';

export default function Game() {
  const [state, dispatch] = useReducer(gameReducer, initialGameState);
  
  const handleStateChange = useCallback((newState) => {
    dispatch({ type: 'SET_GAME_STATE', payload: newState });
  }, []);
  
  // Global keyboard handler for pause/debug
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === 'Escape' && state.gameState === GAME_STATES.PLAYING) {
        handleStateChange(GAME_STATES.PAUSED);
      }
      if (e.key === 'F3') {
        dispatch({ type: 'TOGGLE_DEBUG' });
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [state.gameState, handleStateChange]);
  
  const renderCurrentState = () => {
    switch (state.gameState) {
      case GAME_STATES.MENU:
        return <MainMenu onStateChange={handleStateChange} />;
        
      case GAME_STATES.PLAYING:
        return (
          <>
            <GameCanvas state={state} dispatch={dispatch} />
            {state.debug.enabled && <DebugOverlay state={state} />}
          </>
        );
        
      case GAME_STATES.PAUSED:
        return (
          <>
            <GameCanvas state={state} dispatch={dispatch} frozen />
            <PauseMenu 
              onResume={() => handleStateChange(GAME_STATES.PLAYING)}
              onQuit={() => handleStateChange(GAME_STATES.MENU)}
            />
          </>
        );
        
      case GAME_STATES.LEVEL_EDITOR:
        return <LevelEditor dispatch={dispatch} />;
        
      case GAME_STATES.GAME_OVER:
        return (
          <GameOver 
            score={state.score}
            onRestart={() => {
              dispatch({ type: 'RESET_GAME' });
              handleStateChange(GAME_STATES.PLAYING);
            }}
            onMenu={() => handleStateChange(GAME_STATES.MENU)}
          />
        );
        
      default:
        return <MainMenu onStateChange={handleStateChange} />;
    }
  };
  
  return (
    <div className="game-container" style={{
      width: GAME_CONFIG.SCREEN_WIDTH,
      height: GAME_CONFIG.SCREEN_HEIGHT,
      position: 'relative',
      overflow: 'hidden',
      backgroundColor: '#1a1a2e',
      margin: '0 auto',
    }}>
      {renderCurrentState()}
    </div>
  );
}
```

### Game Loop Hook
```javascript
// hooks/useGameLoop.js
import { useEffect, useRef, useCallback } from 'react';

export function useGameLoop(callback, isRunning = true) {
  const requestRef = useRef();
  const previousTimeRef = useRef();
  const fpsRef = useRef(0);
  const frameCountRef = useRef(0);
  const lastFpsUpdateRef = useRef(0);
  
  const animate = useCallback((time) => {
    if (previousTimeRef.current !== undefined) {
      const deltaTime = time - previousTimeRef.current;
      
      // FPS calculation
      frameCountRef.current++;
      if (time - lastFpsUpdateRef.current >= 1000) {
        fpsRef.current = frameCountRef.current;
        frameCountRef.current = 0;
        lastFpsUpdateRef.current = time;
      }
      
      // Cap deltaTime to prevent spiral of death
      const cappedDelta = Math.min(deltaTime, 33.33); // Max ~30fps worth of time
      
      callback(cappedDelta, fpsRef.current);
    }
    previousTimeRef.current = time;
    requestRef.current = requestAnimationFrame(animate);
  }, [callback]);
  
  useEffect(() => {
    if (isRunning) {
      requestRef.current = requestAnimationFrame(animate);
      return () => cancelAnimationFrame(requestRef.current);
    }
  }, [isRunning, animate]);
  
  return fpsRef.current;
}
```

### Input Hook
```javascript
// hooks/useInput.js
import { useState, useEffect, useCallback } from 'react';

const KEY_MAPPINGS = {
  // Movement
  ArrowLeft: 'left',
  ArrowRight: 'right',
  ArrowUp: 'up',
  ArrowDown: 'down',
  KeyA: 'left',
  KeyD: 'right',
  KeyW: 'up',
  KeyS: 'down',
  
  // Actions
  Space: 'jump',
  KeyZ: 'shoot',
  KeyX: 'kick',
  ShiftLeft: 'roll',
  ShiftRight: 'roll',
  KeyC: 'grenade',
};

export function useInput() {
  const [keys, setKeys] = useState({
    left: false,
    right: false,
    up: false,
    down: false,
    jump: false,
    shoot: false,
    kick: false,
    roll: false,
    grenade: false,
  });
  
  const [justPressed, setJustPressed] = useState({});
  
  const handleKeyDown = useCallback((e) => {
    const action = KEY_MAPPINGS[e.code];
    if (action) {
      e.preventDefault();
      setKeys(prev => {
        if (!prev[action]) {
          setJustPressed(jp => ({ ...jp, [action]: true }));
        }
        return { ...prev, [action]: true };
      });
    }
  }, []);
  
  const handleKeyUp = useCallback((e) => {
    const action = KEY_MAPPINGS[e.code];
    if (action) {
      e.preventDefault();
      setKeys(prev => ({ ...prev, [action]: false }));
    }
  }, []);
  
  // Clear justPressed flags each frame
  const clearJustPressed = useCallback(() => {
    setJustPressed({});
  }, []);
  
  useEffect(() => {
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);
    
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, [handleKeyDown, handleKeyUp]);
  
  return { keys, justPressed, clearJustPressed };
}
```

---

## 5. Player System

### Player Component
```jsx
// Gameplay/Player.jsx
import React, { memo } from 'react';
import { PLAYER_STATES, GAME_CONFIG } from '../constants';
import PlayerSprite from '../sprites/PlayerSprite';

const Player = memo(function Player({ player, cameraX }) {
  const screenX = player.x - cameraX;
  const screenY = player.y;
  
  // Don't render if off-screen
  if (screenX < -player.width || screenX > GAME_CONFIG.SCREEN_WIDTH + player.width) {
    return null;
  }
  
  return (
    <g 
      transform={`translate(${screenX}, ${screenY})`}
      className="player"
    >
      <PlayerSprite 
        state={player.state}
        direction={player.direction}
        frame={player.animationFrame}
        width={player.width}
        height={player.height}
      />
      
      {/* Hitbox debug overlay */}
      {player.debug?.showHitboxes && (
        <rect
          x={0}
          y={0}
          width={player.width}
          height={player.height}
          fill="none"
          stroke="lime"
          strokeWidth={1}
          opacity={0.5}
        />
      )}
    </g>
  );
});

export default Player;
```

### Player SVG Sprite
```jsx
// sprites/PlayerSprite.jsx
import React from 'react';
import { PLAYER_STATES, DIRECTIONS } from '../constants';

// SVG placeholder sprites - 3 frames per animation
const SPRITE_DATA = {
  [PLAYER_STATES.IDLE]: [
    // Frame 1 - Standing
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" /> {/* Head */}
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" /> {/* Body */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" /> {/* Left leg */}
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" /> {/* Right leg */}
      </>
    ),
    // Frame 2 - Breathing
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.18} y={h*0.3} width={w*0.64} height={h*0.42} fill="#228b22" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
    // Frame 3 - Same as 1 (breathing cycle)
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
  ],
  
  [PLAYER_STATES.WALKING]: [
    // Frame 1 - Left leg forward
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.1} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.25} fill="#2f4f4f" />
      </>
    ),
    // Frame 2 - Neutral
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.25} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
    // Frame 3 - Right leg forward
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.25} fill="#2f4f4f" />
        <rect x={w*0.65} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
  ],
  
  [PLAYER_STATES.JUMPING]: [
    // Frame 1-3 - Arms and legs spread
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.35} fill="#228b22" />
        <rect x={w*0.05} y={h*0.65} width={w*0.3} height={h*0.2} fill="#2f4f4f" /> {/* Tucked legs */}
        <rect x={w*0.65} y={h*0.65} width={w*0.3} height={h*0.2} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.35} fill="#228b22" />
        <rect x={w*0.1} y={h*0.6} width={w*0.25} height={h*0.25} fill="#2f4f4f" />
        <rect x={w*0.65} y={h*0.6} width={w*0.25} height={h*0.25} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.35} fill="#228b22" />
        <rect x={w*0.15} y={h*0.65} width={w*0.2} height={h*0.2} fill="#2f4f4f" />
        <rect x={w*0.65} y={h*0.65} width={w*0.2} height={h*0.2} fill="#2f4f4f" />
      </>
    ),
  ],
  
  [PLAYER_STATES.ROLLING]: [
    // Frame 1-3 - Ball shape
    (w, h) => (
      <ellipse cx={w*0.5} cy={h*0.6} rx={w*0.4} ry={h*0.35} fill="#228b22" />
    ),
    (w, h) => (
      <ellipse cx={w*0.5} cy={h*0.6} rx={w*0.45} ry={h*0.3} fill="#228b22" />
    ),
    (w, h) => (
      <ellipse cx={w*0.5} cy={h*0.6} rx={w*0.4} ry={h*0.35} fill="#228b22" />
    ),
  ],
  
  [PLAYER_STATES.SHOOTING]: [
    // Frame 1-3 - Arm extended with gun
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.7} y={h*0.35} width={w*0.4} height={h*0.12} fill="#f4a460" /> {/* Arm */}
        <rect x={w*1.0} y={h*0.32} width={w*0.2} height={h*0.08} fill="#333" /> {/* Gun */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.7} y={h*0.35} width={w*0.35} height={h*0.12} fill="#f4a460" />
        <rect x={w*0.95} y={h*0.32} width={w*0.2} height={h*0.08} fill="#333" />
        <circle cx={w*1.2} cy={h*0.36} r={w*0.08} fill="#ff0" opacity={0.8} /> {/* Muzzle flash */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.7} y={h*0.38} width={w*0.35} height={h*0.12} fill="#f4a460" />
        <rect x={w*0.95} y={h*0.35} width={w*0.2} height={h*0.08} fill="#333" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
  ],
  
  [PLAYER_STATES.KICKING]: [
    // Frame 1-3 - Leg extended
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.15} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.5} y={h*0.55} width={w*0.6} height={h*0.15} fill="#2f4f4f" /> {/* Kicking leg */}
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.1} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.1} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.5} y={h*0.5} width={w*0.7} height={h*0.15} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={0} width={w*0.5} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.15} y={h*0.7} width={w*0.25} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.45} y={h*0.6} width={w*0.5} height={h*0.15} fill="#2f4f4f" />
      </>
    ),
  ],
  
  [PLAYER_STATES.CAPTURED]: [
    // Frame 1-3 - Hands up surrender pose
    (w, h) => (
      <>
        <rect x={w*0.25} y={h*0.05} width={w*0.5} height={h*0.25} fill="#f4a460" />
        <rect x={w*0.1} y={h*0} width={w*0.15} height={h*0.3} fill="#f4a460" /> {/* Left arm up */}
        <rect x={w*0.75} y={h*0} width={w*0.15} height={h*0.3} fill="#f4a460" /> {/* Right arm up */}
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.25} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.25} fill="#f4a460" />
        <rect x={w*0.08} y={h*0.02} width={w*0.15} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.77} y={h*0.02} width={w*0.15} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.32} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.25} y={h*0.72} width={w*0.2} height={h*0.28} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.72} width={w*0.2} height={h*0.28} fill="#2f4f4f" />
      </>
    ),
    (w, h) => (
      <>
        <rect x={w*0.25} y={h*0.05} width={w*0.5} height={h*0.25} fill="#f4a460" />
        <rect x={w*0.1} y={h*0} width={w*0.15} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.75} y={h*0} width={w*0.15} height={h*0.3} fill="#f4a460" />
        <rect x={w*0.2} y={h*0.3} width={w*0.6} height={h*0.4} fill="#228b22" />
        <rect x={w*0.25} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
        <rect x={w*0.55} y={h*0.7} width={w*0.2} height={h*0.3} fill="#2f4f4f" />
      </>
    ),
  ],
};

export default function PlayerSprite({ state, direction, frame, width, height }) {
  const sprites = SPRITE_DATA[state] || SPRITE_DATA[PLAYER_STATES.IDLE];
  const currentFrame = sprites[frame % sprites.length];
  
  return (
    <g transform={direction === DIRECTIONS.LEFT ? `scale(-1, 1) translate(-${width}, 0)` : ''}>
      {currentFrame(width, height)}
    </g>
  );
}
```

### Player Update Logic
```javascript
// systems/playerController.js
import { GAME_CONFIG, PLAYER_STATES, DIRECTIONS } from '../constants';

export function updatePlayer(player, input, platforms, deltaTime) {
  const dt = deltaTime / 16.67; // Normalize to 60fps
  let newPlayer = { ...player };
  
  // Don't update if captured
  if (player.state === PLAYER_STATES.CAPTURED) {
    return updateCapturedState(newPlayer, deltaTime);
  }
  
  // Handle rolling (locked state)
  if (player.state === PLAYER_STATES.ROLLING) {
    return updateRollingState(newPlayer, platforms, deltaTime);
  }
  
  // Horizontal movement
  if (input.keys.left) {
    newPlayer.vx = -GAME_CONFIG.PLAYER_SPEED;
    newPlayer.direction = DIRECTIONS.LEFT;
    if (newPlayer.isGrounded) {
      newPlayer.state = PLAYER_STATES.WALKING;
    }
  } else if (input.keys.right) {
    newPlayer.vx = GAME_CONFIG.PLAYER_SPEED;
    newPlayer.direction = DIRECTIONS.RIGHT;
    if (newPlayer.isGrounded) {
      newPlayer.state = PLAYER_STATES.WALKING;
    }
  } else {
    newPlayer.vx *= GAME_CONFIG.FRICTION;
    if (Math.abs(newPlayer.vx) < 0.1) newPlayer.vx = 0;
    if (newPlayer.isGrounded && newPlayer.state === PLAYER_STATES.WALKING) {
      newPlayer.state = PLAYER_STATES.IDLE;
    }
  }
  
  // Jumping
  if (input.justPressed.jump && newPlayer.isGrounded) {
    newPlayer.vy = GAME_CONFIG.PLAYER_JUMP_FORCE;
    newPlayer.isGrounded = false;
    newPlayer.state = PLAYER_STATES.JUMPING;
  }
  
  // Rolling
  if (input.justPressed.roll && newPlayer.isGrounded) {
    newPlayer.state = PLAYER_STATES.ROLLING;
    newPlayer.rollTimer = GAME_CONFIG.PLAYER_ROLL_DURATION;
    newPlayer.vx = newPlayer.direction * GAME_CONFIG.PLAYER_ROLL_SPEED;
  }
  
  // Shooting
  if (input.keys.shoot) {
    if (newPlayer.isGrounded) {
      newPlayer.state = PLAYER_STATES.SHOOTING;
    }
    // Actual bullet creation handled separately
  }
  
  // Kicking
  if (input.justPressed.kick && newPlayer.isGrounded) {
    newPlayer.state = PLAYER_STATES.KICKING;
    newPlayer.kickTimer = 300; // ms
  }
  
  // Apply gravity
  newPlayer.vy += GAME_CONFIG.GRAVITY * dt;
  newPlayer.vy = Math.min(newPlayer.vy, GAME_CONFIG.MAX_FALL_SPEED);
  
  // Air state
  if (!newPlayer.isGrounded) {
    if (newPlayer.vy > 0) {
      newPlayer.state = PLAYER_STATES.FALLING;
    } else if (newPlayer.state !== PLAYER_STATES.JUMPING) {
      newPlayer.state = PLAYER_STATES.JUMPING;
    }
  }
  
  // Apply velocity
  newPlayer.x += newPlayer.vx * dt;
  newPlayer.y += newPlayer.vy * dt;
  
  // Platform collision
  newPlayer = resolvePlayerPlatformCollision(newPlayer, platforms);
  
  // Animation frame update
  newPlayer.animationTimer = (newPlayer.animationTimer || 0) + deltaTime;
  if (newPlayer.animationTimer >= GAME_CONFIG.FRAME_DURATION) {
    newPlayer.animationFrame = (newPlayer.animationFrame + 1) % GAME_CONFIG.SPRITE_FRAMES;
    newPlayer.animationTimer = 0;
  }
  
  // Update kick timer
  if (newPlayer.kickTimer > 0) {
    newPlayer.kickTimer -= deltaTime;
    if (newPlayer.kickTimer <= 0) {
      newPlayer.state = PLAYER_STATES.IDLE;
    }
  }
  
  // Invulnerability timer
  if (newPlayer.invulnerable) {
    newPlayer.invulnerableTimer -= deltaTime;
    if (newPlayer.invulnerableTimer <= 0) {
      newPlayer.invulnerable = false;
    }
  }
  
  return newPlayer;
}

function updateRollingState(player, platforms, deltaTime) {
  const dt = deltaTime / 16.67;
  let newPlayer = { ...player };
  
  newPlayer.rollTimer -= deltaTime;
  
  // Continue roll movement
  newPlayer.x += newPlayer.vx * dt;
  
  // Apply gravity during roll
  newPlayer.vy += GAME_CONFIG.GRAVITY * dt;
  newPlayer.y += newPlayer.vy * dt;
  
  // Collision
  newPlayer = resolvePlayerPlatformCollision(newPlayer, platforms);
  
  // End roll
  if (newPlayer.rollTimer <= 0) {
    newPlayer.state = PLAYER_STATES.IDLE;
    newPlayer.vx *= 0.5; // Slow down after roll
  }
  
  // Animation
  newPlayer.animationTimer = (newPlayer.animationTimer || 0) + deltaTime;
  if (newPlayer.animationTimer >= GAME_CONFIG.FRAME_DURATION * 0.5) {
    newPlayer.animationFrame = (newPlayer.animationFrame + 1) % GAME_CONFIG.SPRITE_FRAMES;
    newPlayer.animationTimer = 0;
  }
  
  return newPlayer;
}

function updateCapturedState(player, deltaTime) {
  // Just animate the captured pose
  let newPlayer = { ...player };
  
  newPlayer.animationTimer = (newPlayer.animationTimer || 0) + deltaTime;
  if (newPlayer.animationTimer >= GAME_CONFIG.FRAME_DURATION * 2) {
    newPlayer.animationFrame = (newPlayer.animationFrame + 1) % GAME_CONFIG.SPRITE_FRAMES;
    newPlayer.animationTimer = 0;
  }
  
  return newPlayer;
}

function resolvePlayerPlatformCollision(player, platforms) {
  let newPlayer = { ...player };
  newPlayer.isGrounded = false;
  
  for (const platform of platforms) {
    // AABB collision check
    if (
      newPlayer.x < platform.x + platform.width &&
      newPlayer.x + newPlayer.width > platform.x &&
      newPlayer.y < platform.y + platform.height &&
      newPlayer.y + newPlayer.height > platform.y
    ) {
      // Determine collision side
      const overlapLeft = (newPlayer.x + newPlayer.width) - platform.x;
      const overlapRight = (platform.x + platform.width) - newPlayer.x;
      const overlapTop = (newPlayer.y + newPlayer.height) - platform.y;
      const overlapBottom = (platform.y + platform.height) - newPlayer.y;
      
      const minOverlapX = Math.min(overlapLeft, overlapRight);
      const minOverlapY = Math.min(overlapTop, overlapBottom);
      
      if (minOverlapY < minOverlapX) {
        // Vertical collision
        if (overlapTop < overlapBottom) {
          // Landing on top
          newPlayer.y = platform.y - newPlayer.height;
          newPlayer.vy = 0;
          newPlayer.isGrounded = true;
        } else {
          // Hitting from below
          newPlayer.y = platform.y + platform.height;
          newPlayer.vy = 0;
        }
      } else {
        // Horizontal collision
        if (overlapLeft < overlapRight) {
          newPlayer.x = platform.x - newPlayer.width;
        } else {
          newPlayer.x = platform.x + platform.width;
        }
        newPlayer.vx = 0;
      }
    }
  }
  
  return newPlayer;
}

// Handle player being hit
export function handlePlayerHit(player) {
  return {
    ...player,
    state: PLAYER_STATES.CAPTURED,
    vx: 0,
    vy: 0,
    animationFrame: 0,
  };
}
```

---

## 6. Enemy System

### Enemy Component
```jsx
// Gameplay/Enemy.jsx
import React, { memo } from 'react';
import { ENEMY_STATES, GAME_CONFIG } from '../constants';
import EnemySprite from '../sprites/EnemySprite';

const Enemy = memo(function Enemy({ enemy, cameraX }) {
  const screenX = enemy.x - cameraX;
  const screenY = enemy.y;
  
  // Don't render if off-screen (with buffer for spawning)
  if (screenX < -100 || screenX > GAME_CONFIG.SCREEN_WIDTH + 100) {
    return null;
  }
  
  return (
    <g 
      transform={`translate(${screenX}, ${screenY})`}
      className="enemy"
      opacity={enemy.state === ENEMY_STATES.DYING ? 0.5 : 1}
    >
      <EnemySprite 
        state={enemy.state}
        direction={enemy.direction}
        frame={enemy.animationFrame}
        width={enemy.width}
        height={enemy.height}
        type={enemy.type}
      />
      
      {/* Health bar */}
      {enemy.health < enemy.maxHealth && enemy.state !== ENEMY_STATES.DEAD && (
        <g transform={`translate(0, -8)`}>
          <rect x={0} y={0} width={enemy.width} height={4} fill="#333" />
          <rect 
            x={0} 
            y={0} 
            width={enemy.width * (enemy.health / enemy.maxHealth)} 
            height={4} 
            fill="#e74c3c" 
          />
        </g>
      )}
    </g>
  );
});

export default Enemy;
```

### Enemy SVG Sprite
```jsx
// sprites/EnemySprite.jsx
import React from 'react';
import { ENEMY_STATES, DIRECTIONS } from '../constants';

// SVG placeholder sprites for "goon" enemies
const ENEMY_SPRITE_DATA = {
  [ENEMY_STATES.IDLE]: [
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" /> {/* Helmet */}
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" /> {/* Head */}
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" /> {/* Body */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" /> {/* Legs */}
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.13} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.09} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.31} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.71} width={w*0.25} height={h*0.29} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.71} width={w*0.25} height={h*0.29} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
  ],
  
  [ENEMY_STATES.PATROLLING]: [
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.1} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.25} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.25} y={h*0.7} width={w*0.2} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.2} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.25} fill="#5d4e37" />
        <rect x={w*0.65} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
  ],
  
  [ENEMY_STATES.ATTACKING]: [
    // Punch wind-up
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#8b4513" />
        <rect x={w*-0.1} y={h*0.35} width={w*0.3} height={h*0.12} fill="#d4a574" /> {/* Arm back */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    // Punch mid
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.6} y={h*0.38} width={w*0.4} height={h*0.12} fill="#d4a574" /> {/* Arm forward */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    // Punch extended
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.55} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.65} y={h*0.35} width={w*0.5} height={h*0.15} fill="#d4a574" /> {/* Full punch */}
        <circle cx={w*1.1} cy={h*0.42} r={w*0.12} fill="#d4a574" /> {/* Fist */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
  ],
  
  [ENEMY_STATES.SHOOTING]: [
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.65} y={h*0.35} width={w*0.35} height={h*0.1} fill="#d4a574" /> {/* Arm */}
        <rect x={w*0.9} y={h*0.32} width={w*0.25} height={h*0.08} fill="#2c3e50" /> {/* Gun */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.65} y={h*0.35} width={w*0.35} height={h*0.1} fill="#d4a574" />
        <rect x={w*0.9} y={h*0.32} width={w*0.25} height={h*0.08} fill="#2c3e50" />
        <circle cx={w*1.2} cy={h*0.36} r={w*0.1} fill="#ff6600" opacity={0.9} /> {/* Muzzle flash */}
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.6} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.65} y={h*0.38} width={w*0.35} height={h*0.1} fill="#d4a574" />
        <rect x={w*0.9} y={h*0.35} width={w*0.25} height={h*0.08} fill="#2c3e50" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </>
    ),
  ],
  
  [ENEMY_STATES.HIT]: [
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.15} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.11} width={w*0.5} height={h*0.22} fill="#e07050" /> {/* Reddish from hit */}
        <rect x={w*0.2} y={h*0.33} width={w*0.7} height={h*0.38} fill="#8b4513" />
        <rect x={w*0.25} y={h*0.7} width={w*0.2} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.2} height={h*0.3} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.45} cy={h*0.18} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.2} y={h*0.14} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.36} width={w*0.7} height={h*0.38} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.72} width={w*0.25} height={h*0.28} fill="#5d4e37" />
        <rect x={w*0.5} y={h*0.72} width={w*0.25} height={h*0.28} fill="#5d4e37" />
      </>
    ),
    (w, h) => (
      <>
        <ellipse cx={w*0.5} cy={h*0.15} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.11} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.33} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.22} y={h*0.71} width={w*0.23} height={h*0.29} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.71} width={w*0.23} height={h*0.29} fill="#5d4e37" />
      </>
    ),
  ],
  
  [ENEMY_STATES.DYING]: [
    // Falling backward
    (w, h) => (
      <g transform={`rotate(-20, ${w*0.5}, ${h*0.5})`}>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </g>
    ),
    (w, h) => (
      <g transform={`rotate(-45, ${w*0.5}, ${h*0.5})`}>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </g>
    ),
    (w, h) => (
      <g transform={`rotate(-80, ${w*0.5}, ${h*0.5}) translate(0, ${h*0.2})`}>
        <ellipse cx={w*0.5} cy={h*0.12} rx={w*0.35} ry={h*0.12} fill="#1e3a5f" />
        <rect x={w*0.25} y={h*0.08} width={w*0.5} height={h*0.22} fill="#d4a574" />
        <rect x={w*0.15} y={h*0.3} width={w*0.7} height={h*0.4} fill="#8b4513" />
        <rect x={w*0.2} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
        <rect x={w*0.55} y={h*0.7} width={w*0.25} height={h*0.3} fill="#5d4e37" />
      </g>
    ),
  ],
  
  [ENEMY_STATES.DEAD]: [
    // Lying on ground
    (w, h) => (
      <g transform={`translate(0, ${h*0.5})`}>
        <rect x={w*0.1} y={h*0.3} width={h*0.8} height={w*0.4} fill="#8b4513" />
        <ellipse cx={w*0.05} cy={h*0.4} rx={h*0.12} ry={w*0.18} fill="#d4a574" />
        <ellipse cx={w*0.95} cy={h*0.45} rx={h*0.08} ry={w*0.12} fill="#5d4e37" />
      </g>
    ),
    (w, h) => (
      <g transform={`translate(0, ${h*0.5})`}>
        <rect x={w*0.1} y={h*0.3} width={h*0.8} height={w*0.4} fill="#8b4513" />
        <ellipse cx={w*0.05} cy={h*0.4} rx={h*0.12} ry={w*0.18} fill="#d4a574" />
        <ellipse cx={w*0.95} cy={h*0.45} rx={h*0.08} ry={w*0.12} fill="#5d4e37" />
      </g>
    ),
    (w, h) => (
      <g transform={`translate(0, ${h*0.5})`}>
        <rect x={w*0.1} y={h*0.3} width={h*0.8} height={w*0.4} fill="#8b4513" />
        <ellipse cx={w*0.05} cy={h*0.4} rx={h*0.12} ry={w*0.18} fill="#d4a574" />
        <ellipse cx={w*0.95} cy={h*0.45} rx={h*0.08} ry={w*0.12} fill="#5d4e37" />
      </g>
    ),
  ],
};

export default function EnemySprite({ state, direction, frame, width, height, type }) {
  const sprites = ENEMY_SPRITE_DATA[state] || ENEMY_SPRITE_DATA[ENEMY_STATES.IDLE];
  const currentFrame = sprites[frame % sprites.length];
  
  return (
    <g transform={direction === DIRECTIONS.LEFT ? `scale(-1, 1) translate(-${width}, 0)` : ''}>
      {currentFrame(width, height)}
    </g>
  );
}
```

### Enemy Update Logic
```javascript
// systems/enemyController.js
import { GAME_CONFIG, ENEMY_STATES, DIRECTIONS } from '../constants';

export function createEnemy(x, y, type = 'goon') {
  return {
    id: `enemy_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
    type,
    x,
    y,
    vx: 0,
    vy: 0,
    width: 28,
    height: 44,
    direction: DIRECTIONS.LEFT,
    state: ENEMY_STATES.IDLE,
    health: 100,
    maxHealth: 100,
    damage: 20,
    animationFrame: 0,
    animationTimer: 0,
    
    // Behavior
    patrolDistance: 100,
    patrolOrigin: x,
    aggroRange: GAME_CONFIG.ENEMY_AGGRO_RANGE,
    attackRange: GAME_CONFIG.ENEMY_ATTACK_RANGE,
    shootRange: GAME_CONFIG.ENEMY_SHOOT_RANGE,
    
    // Timers
    attackCooldown: 0,
    shootCooldown: 0,
    stateTimer: 0,
    dyingTimer: 0,
  };
}

export function updateEnemy(enemy, player, platforms, deltaTime) {
  const dt = deltaTime / 16.67;
  let newEnemy = { ...enemy };
  
  // Update animation
  newEnemy.animationTimer += deltaTime;
  if (newEnemy.animationTimer >= GAME_CONFIG.FRAME_DURATION) {
    newEnemy.animationFrame = (newEnemy.animationFrame + 1) % GAME_CONFIG.SPRITE_FRAMES;
    newEnemy.animationTimer = 0;
  }
  
  // Update cooldowns
  if (newEnemy.attackCooldown > 0) newEnemy.attackCooldown -= deltaTime;
  if (newEnemy.shootCooldown > 0) newEnemy.shootCooldown -= deltaTime;
  
  // State machine
  switch (newEnemy.state) {
    case ENEMY_STATES.IDLE:
      newEnemy = updateIdleState(newEnemy, player, deltaTime);
      break;
      
    case ENEMY_STATES.PATROLLING:
      newEnemy = updatePatrolState(newEnemy, player, deltaTime);
      break;
      
    case ENEMY_STATES.CHASING:
      newEnemy = updateChaseState(newEnemy, player, deltaTime);
      break;
      
    case ENEMY_STATES.ATTACKING:
      newEnemy = updateAttackState(newEnemy, player, deltaTime);
      break;
      
    case ENEMY_STATES.SHOOTING:
      newEnemy = updateShootState(newEnemy, player, deltaTime);
      break;
      
    case ENEMY_STATES.HIT:
      newEnemy = updateHitState(newEnemy, deltaTime);
      break;
      
    case ENEMY_STATES.DYING:
      newEnemy = updateDyingState(newEnemy, deltaTime);
      break;
      
    case ENEMY_STATES.DEAD:
      // Do nothing, wait for cleanup
      break;
  }
  
  // Apply physics (if not dead)
  if (newEnemy.state !== ENEMY_STATES.DEAD) {
    newEnemy.vy += GAME_CONFIG.GRAVITY * dt;
    newEnemy.vy = Math.min(newEnemy.vy, GAME_CONFIG.MAX_FALL_SPEED);
    
    newEnemy.x += newEnemy.vx * dt;
    newEnemy.y += newEnemy.vy * dt;
    
    // Platform collision
    newEnemy = resolveEnemyPlatformCollision(newEnemy, platforms);
  }
  
  return newEnemy;
}

function updateIdleState(enemy, player, deltaTime) {
  const distToPlayer = getDistanceToPlayer(enemy, player);
  
  // Check for player detection
  if (distToPlayer < enemy.aggroRange) {
    return {
      ...enemy,
      state: ENEMY_STATES.CHASING,
      direction: player.x < enemy.x ? DIRECTIONS.LEFT : DIRECTIONS.RIGHT,
    };
  }
  
  // Random chance to start patrolling
  enemy.stateTimer += deltaTime;
  if (enemy.stateTimer > 2000) {
    enemy.stateTimer = 0;
    if (Math.random() < 0.3) {
      return {
        ...enemy,
        state: ENEMY_STATES.PATROLLING,
        direction: Math.random() < 0.5 ? DIRECTIONS.LEFT : DIRECTIONS.RIGHT,
      };
    }
  }
  
  return enemy;
}

function updatePatrolState(enemy, player, deltaTime) {
  let newEnemy = { ...enemy };
  const distToPlayer = getDistanceToPlayer(enemy, player);
  
  // Check for player detection
  if (distToPlayer < enemy.aggroRange) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.CHASING,
      direction: player.x < enemy.x ? DIRECTIONS.LEFT : DIRECTIONS.RIGHT,
      vx: 0,
    };
  }
  
  // Move in patrol direction
  newEnemy.vx = GAME_CONFIG.ENEMY_SPEED * newEnemy.direction;
  
  // Check patrol boundaries
  const distFromOrigin = newEnemy.x - newEnemy.patrolOrigin;
  if (Math.abs(distFromOrigin) > enemy.patrolDistance) {
    newEnemy.direction *= -1;
    newEnemy.vx = GAME_CONFIG.ENEMY_SPEED * newEnemy.direction;
  }
  
  // Random chance to stop
  newEnemy.stateTimer += deltaTime;
  if (newEnemy.stateTimer > 3000 && Math.random() < 0.1) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.IDLE,
      stateTimer: 0,
      vx: 0,
    };
  }
  
  return newEnemy;
}

function updateChaseState(enemy, player, deltaTime) {
  let newEnemy = { ...enemy };
  const distToPlayer = getDistanceToPlayer(enemy, player);
  
  // Face player
  newEnemy.direction = player.x < enemy.x ? DIRECTIONS.LEFT : DIRECTIONS.RIGHT;
  
  // Check attack range (punch)
  if (distToPlayer < enemy.attackRange && enemy.attackCooldown <= 0) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.ATTACKING,
      stateTimer: 0,
      vx: 0,
    };
  }
  
  // Check shoot range
  if (distToPlayer > enemy.attackRange && distToPlayer < enemy.shootRange && enemy.shootCooldown <= 0) {
    // 30% chance to shoot when in range
    if (Math.random() < 0.3) {
      return {
        ...newEnemy,
        state: ENEMY_STATES.SHOOTING,
        stateTimer: 0,
        vx: 0,
      };
    }
  }
  
  // Move toward player
  if (distToPlayer > enemy.attackRange * 0.8) {
    newEnemy.vx = GAME_CONFIG.ENEMY_SPEED * 1.5 * newEnemy.direction;
  } else {
    newEnemy.vx = 0;
  }
  
  // Lose aggro if too far
  if (distToPlayer > enemy.aggroRange * 1.5) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.IDLE,
      vx: 0,
    };
  }
  
  return newEnemy;
}

function updateAttackState(enemy, player, deltaTime) {
  let newEnemy = { ...enemy };
  newEnemy.stateTimer += deltaTime;
  
  // Attack animation: 3 frames * FRAME_DURATION = ~450ms
  const attackDuration = GAME_CONFIG.FRAME_DURATION * 3;
  
  if (newEnemy.stateTimer >= attackDuration) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.CHASING,
      stateTimer: 0,
      attackCooldown: 800, // Cooldown before next attack
      animationFrame: 0,
    };
  }
  
  return newEnemy;
}

function updateShootState(enemy, player, deltaTime) {
  let newEnemy = { ...enemy };
  newEnemy.stateTimer += deltaTime;
  
  const shootDuration = GAME_CONFIG.FRAME_DURATION * 3;
  
  // Bullet spawned at frame 1 (handled by game loop)
  
  if (newEnemy.stateTimer >= shootDuration) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.CHASING,
      stateTimer: 0,
      shootCooldown: GAME_CONFIG.ENEMY_FIRE_RATE,
      animationFrame: 0,
    };
  }
  
  return newEnemy;
}

function updateHitState(enemy, deltaTime) {
  let newEnemy = { ...enemy };
  newEnemy.stateTimer += deltaTime;
  
  // Stagger for 300ms
  if (newEnemy.stateTimer >= 300) {
    if (newEnemy.health <= 0) {
      return {
        ...newEnemy,
        state: ENEMY_STATES.DYING,
        stateTimer: 0,
        dyingTimer: 0,
      };
    }
    return {
      ...newEnemy,
      state: ENEMY_STATES.CHASING,
      stateTimer: 0,
    };
  }
  
  return newEnemy;
}

function updateDyingState(enemy, deltaTime) {
  let newEnemy = { ...enemy };
  newEnemy.dyingTimer += deltaTime;
  
  // Death animation ~500ms
  if (newEnemy.dyingTimer >= 500) {
    return {
      ...newEnemy,
      state: ENEMY_STATES.DEAD,
    };
  }
  
  return newEnemy;
}

function getDistanceToPlayer(enemy, player) {
  const dx = player.x + player.width / 2 - (enemy.x + enemy.width / 2);
  const dy = player.y + player.height / 2 - (enemy.y + enemy.height / 2);
  return Math.sqrt(dx * dx + dy * dy);
}

function resolveEnemyPlatformCollision(enemy, platforms) {
  let newEnemy = { ...enemy };
  
  for (const platform of platforms) {
    if (
      newEnemy.x < platform.x + platform.width &&
      newEnemy.x + newEnemy.width > platform.x &&
      newEnemy.y < platform.y + platform.height &&
      newEnemy.y + newEnemy.height > platform.y
    ) {
      const overlapTop = (newEnemy.y + newEnemy.height) - platform.y;
      const overlapBottom = (platform.y + platform.height) - newEnemy.y;
      
      if (overlapTop < overlapBottom && newEnemy.vy > 0) {
        newEnemy.y = platform.y - newEnemy.height;
        newEnemy.vy = 0;
      }
    }
  }
  
  return newEnemy;
}

export function handleEnemyHit(enemy, damage) {
  const newHealth = enemy.health - damage;
  
  return {
    ...enemy,
    health: newHealth,
    state: ENEMY_STATES.HIT,
    stateTimer: 0,
    animationFrame: 0,
    vx: 0,
  };
}
```

---

## 7. Collision System

### Collision Detection
```javascript
// systems/collision.js
import { PLAYER_STATES, ENEMY_STATES, GAME_CONFIG } from '../constants';

// AABB collision check
export function checkAABBCollision(a, b) {
  return (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  );
}

// Check if player kick hits enemy
export function checkKickCollision(player, enemy) {
  if (player.state !== PLAYER_STATES.KICKING) return false;
  
  const kickBox = {
    x: player.direction > 0 
      ? player.x + player.width 
      : player.x - GAME_CONFIG.KICK_RANGE,
    y: player.y + player.height * 0.3,
    width: GAME_CONFIG.KICK_RANGE,
    height: player.height * 0.4,
  };
  
  return checkAABBCollision(kickBox, enemy);
}

// Check bullet collisions
export function checkBulletCollisions(bullets, targets) {
  const results = [];
  
  bullets.forEach((bullet, bulletIndex) => {
    targets.forEach((target, targetIndex) => {
      if (checkAABBCollision(bullet, target)) {
        results.push({
          bulletIndex,
          targetIndex,
          bullet,
          target,
        });
      }
    });
  });
  
  return results;
}

// Check enemy attack hits player
export function checkEnemyAttackHit(enemy, player) {
  if (enemy.state !== ENEMY_STATES.ATTACKING) return false;
  if (player.state === PLAYER_STATES.ROLLING) return false; // Roll invincibility
  if (player.invulnerable) return false;
  
  // Only check on last frame of attack
  if (enemy.animationFrame !== 2) return false;
  
  const attackBox = {
    x: enemy.direction > 0 
      ? enemy.x + enemy.width 
      : enemy.x - GAME_CONFIG.ENEMY_ATTACK_RANGE,
    y: enemy.y + enemy.height * 0.2,
    width: GAME_CONFIG.ENEMY_ATTACK_RANGE,
    height: enemy.height * 0.5,
  };
  
  return checkAABBCollision(attackBox, player);
}

// Main collision handler for game loop
export function processCollisions(state) {
  const { player, enemies, bullets, level } = state;
  const collisionEvents = [];
  
  // Player bullets hitting enemies
  const playerBullets = bullets.filter(b => b.owner === 'player');
  const enemyHits = checkBulletCollisions(
    playerBullets, 
    enemies.filter(e => e.state !== ENEMY_STATES.DEAD && e.state !== ENEMY_STATES.DYING)
  );
  
  enemyHits.forEach(hit => {
    collisionEvents.push({
      type: 'BULLET_HIT_ENEMY',
      bulletIndex: bullets.indexOf(hit.bullet),
      enemyIndex: hit.targetIndex,
      damage: hit.bullet.damage || 25,
    });
  });
  
  // Enemy bullets hitting player
  const enemyBullets = bullets.filter(b => b.owner === 'enemy');
  const playerHits = checkBulletCollisions(enemyBullets, [player]);
  
  if (playerHits.length > 0 && player.state !== PLAYER_STATES.ROLLING && !player.invulnerable) {
    collisionEvents.push({
      type: 'BULLET_HIT_PLAYER',
      bulletIndex: bullets.indexOf(playerHits[0].bullet),
    });
  }
  
  // Player kick hitting enemies
  enemies.forEach((enemy, index) => {
    if (enemy.state !== ENEMY_STATES.DEAD && enemy.state !== ENEMY_STATES.DYING) {
      if (checkKickCollision(player, enemy)) {
        collisionEvents.push({
          type: 'KICK_HIT_ENEMY',
          enemyIndex: index,
          damage: GAME_CONFIG.KICK_DAMAGE,
        });
      }
    }
  });
  
  // Enemy attacks hitting player
  enemies.forEach((enemy, index) => {
    if (checkEnemyAttackHit(enemy, player)) {
      collisionEvents.push({
        type: 'ENEMY_ATTACK_HIT_PLAYER',
        enemyIndex: index,
      });
    }
  });
  
  // Player reaching end zone
  if (checkAABBCollision(player, level.endZone)) {
    collisionEvents.push({ type: 'LEVEL_COMPLETE' });
  }
  
  return collisionEvents;
}
```

### Bullet Component
```jsx
// Gameplay/Bullet.jsx
import React, { memo } from 'react';

const Bullet = memo(function Bullet({ bullet, cameraX }) {
  const screenX = bullet.x - cameraX;
  const screenY = bullet.y;
  
  // Skip if off-screen
  if (screenX < -20 || screenX > 820) return null;
  
  const isPlayerBullet = bullet.owner === 'player';
  
  return (
    <g transform={`translate(${screenX}, ${screenY})`}>
      {isPlayerBullet ? (
        // Player bullet - yellow/orange
        <>
          <ellipse 
            cx={bullet.width / 2} 
            cy={bullet.height / 2} 
            rx={bullet.width / 2} 
            ry={bullet.height / 2} 
            fill="#ffcc00"
          />
          <ellipse 
            cx={bullet.width / 2} 
            cy={bullet.height / 2} 
            rx={bullet.width / 3} 
            ry={bullet.height / 3} 
            fill="#ffffff"
          />
        </>
      ) : (
        // Enemy bullet - red
        <>
          <circle 
            cx={bullet.width / 2} 
            cy={bullet.height / 2} 
            r={bullet.width / 2} 
            fill="#ff3333"
          />
          <circle 
            cx={bullet.width / 2} 
            cy={bullet.height / 2} 
            r={bullet.width / 3} 
            fill="#ff9999"
          />
        </>
      )}
    </g>
  );
});

export default Bullet;

// Bullet creation helpers
export function createPlayerBullet(player) {
  const bulletX = player.direction > 0 
    ? player.x + player.width 
    : player.x - 8;
  
  return {
    id: `bullet_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
    x: bulletX,
    y: player.y + player.height * 0.35,
    vx: GAME_CONFIG.BULLET_SPEED * player.direction,
    vy: 0,
    width: 8,
    height: 4,
    owner: 'player',
    damage: 25,
  };
}

export function createEnemyBullet(enemy) {
  const bulletX = enemy.direction > 0 
    ? enemy.x + enemy.width 
    : enemy.x - 6;
  
  return {
    id: `bullet_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
    x: bulletX,
    y: enemy.y + enemy.height * 0.35,
    vx: GAME_CONFIG.BULLET_SPEED * 0.6 * enemy.direction,
    vy: 0,
    width: 6,
    height: 6,
    owner: 'enemy',
    damage: 20,
  };
}

export function updateBullets(bullets, levelWidth, deltaTime) {
  const dt = deltaTime / 16.67;
  
  return bullets
    .map(bullet => ({
      ...bullet,
      x: bullet.x + bullet.vx * dt,
      y: bullet.y + bullet.vy * dt,
    }))
    .filter(bullet => 
      bullet.x > -50 && 
      bullet.x < levelWidth + 50 &&
      bullet.y > -50 &&
      bullet.y < 500
    );
}
```

---

## 8. Level System

### Level Data Structure
```javascript
// data/levels/level1.js
export const level1 = {
  id: 1,
  name: "Training Grounds",
  width: 3200,
  height: 450,
  
  // Background layers (parallax)
  backgrounds: [
    { image: 'sky', scrollFactor: 0.1 },
    { image: 'mountains', scrollFactor: 0.3 },
    { image: 'trees', scrollFactor: 0.6 },
  ],
  
  // Ground and platforms
  platforms: [
    // Main ground
    { x: 0, y: 400, width: 800, height: 50, type: 'ground' },
    { x: 800, y: 400, width: 400, height: 50, type: 'ground' },
    { x: 1200, y: 400, width: 600, height: 50, type: 'ground' },
    { x: 1800, y: 400, width: 400, height: 50, type: 'ground' },
    { x: 2200, y: 400, width: 1000, height: 50, type: 'ground' },
    
    // Elevated platforms
    { x: 400, y: 320, width: 150, height: 20, type: 'platform' },
    { x: 700, y: 280, width: 100, height: 20, type: 'platform' },
    { x: 1400, y: 300, width: 200, height: 20, type: 'platform' },
    { x: 2400, y: 320, width: 150, height: 20, type: 'platform' },
    { x: 2700, y: 280, width: 100, height: 20, type: 'platform' },
  ],
  
  // Player spawn point
  playerSpawn: { x: 100, y: 350 },
  
  // End zone (level complete trigger)
  endZone: { x: 3100, y: 0, width: 100, height: 450 },
  
  // Enemy spawn points
  enemies: [
    { x: 400, y: 350, type: 'goon' },
    { x: 600, y: 350, type: 'goon' },
    { x: 900, y: 350, type: 'goon' },
    { x: 1300, y: 350, type: 'goon' },
    { x: 1500, y: 250, type: 'goon' }, // On platform
    { x: 1700, y: 350, type: 'goon' },
    { x: 2000, y: 350, type: 'goon' },
    { x: 2300, y: 350, type: 'goon' },
    { x: 2500, y: 270, type: 'goon' }, // On platform
    { x: 2800, y: 350, type: 'goon' },
    { x: 2900, y: 350, type: 'goon' },
  ],
  
  // Pickups and items (future feature)
  items: [
    { x: 420, y: 280, type: 'ammo' },
    { x: 1420, y: 260, type: 'health' },
    { x: 2420, y: 280, type: 'weapon_hmg' },
  ],
};
```

### Level Loader
```javascript
// systems/levelLoader.js
import { createEnemy } from './enemyController';
import { level1 } from '../data/levels/level1';

const LEVELS = {
  1: level1,
  // Add more levels here
};

export function loadLevel(levelNumber) {
  const levelData = LEVELS[levelNumber];
  
  if (!levelData) {
    console.error(`Level ${levelNumber} not found`);
    return null;
  }
  
  // Create enemy instances from spawn points
  const enemies = levelData.enemies.map(spawn => 
    createEnemy(spawn.x, spawn.y, spawn.type)
  );
  
  return {
    level: {
      id: levelData.id,
      name: levelData.name,
      width: levelData.width,
      height: levelData.height,
      platforms: levelData.platforms,
      backgrounds: levelData.backgrounds,
      endZone: levelData.endZone,
      items: levelData.items || [],
    },
    playerSpawn: levelData.playerSpawn,
    enemies,
  };
}

export function saveLevelToJSON(levelData) {
  return JSON.stringify(levelData, null, 2);
}

export function loadLevelFromJSON(jsonString) {
  try {
    return JSON.parse(jsonString);
  } catch (e) {
    console.error('Failed to parse level JSON:', e);
    return null;
  }
}
```

### Camera System
```javascript
// hooks/useCamera.js
import { useCallback } from 'react';
import { GAME_CONFIG } from '../constants';

export function useCamera() {
  const updateCamera = useCallback((camera, player, levelWidth) => {
    // Target X: Center player on screen with deadzone
    const screenCenter = GAME_CONFIG.SCREEN_WIDTH / 2;
    const deadzoneWidth = 100;
    
    let targetX = camera.x;
    const playerScreenX = player.x - camera.x;
    
    // Push camera if player leaves deadzone
    if (playerScreenX > screenCenter + deadzoneWidth) {
      targetX = player.x - screenCenter - deadzoneWidth;
    } else if (playerScreenX < screenCenter - deadzoneWidth) {
      targetX = player.x - screenCenter + deadzoneWidth;
    }
    
    // Clamp to level bounds
    targetX = Math.max(0, Math.min(targetX, levelWidth - GAME_CONFIG.SCREEN_WIDTH));
    
    // Smooth camera movement
    const smoothing = 0.1;
    const newX = camera.x + (targetX - camera.x) * smoothing;
    
    // Apply screen shake
    let shakeX = 0;
    let shakeY = 0;
    if (camera.shakeIntensity > 0) {
      shakeX = (Math.random() - 0.5) * camera.shakeIntensity;
      shakeY = (Math.random() - 0.5) * camera.shakeIntensity;
    }
    
    return {
      x: newX + shakeX,
      y: camera.y + shakeY,
      shakeIntensity: Math.max(0, camera.shakeIntensity - 0.5),
    };
  }, []);
  
  const addScreenShake = useCallback((intensity) => {
    return { shakeIntensity: intensity };
  }, []);
  
  return { updateCamera, addScreenShake };
}
```

### Background Component
```jsx
// Gameplay/Background.jsx
import React, { memo } from 'react';
import { GAME_CONFIG } from '../constants';

const Background = memo(function Background({ cameraX, levelWidth }) {
  // Parallax layers
  const layers = [
    { color: '#87CEEB', scrollFactor: 0 }, // Sky (static)
    { color: '#4a6fa5', scrollFactor: 0.2 }, // Far mountains
    { color: '#2d4a6f', scrollFactor: 0.4 }, // Near mountains
    { color: '#1a3a5c', scrollFactor: 0.6 }, // Hills
  ];
  
  return (
    <g className="background">
      {/* Sky gradient */}
      <defs>
        <linearGradient id="skyGradient" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" style={{ stopColor: '#1a1a2e', stopOpacity: 1 }} />
          <stop offset="50%" style={{ stopColor: '#16213e', stopOpacity: 1 }} />
          <stop offset="100%" style={{ stopColor: '#1f4068', stopOpacity: 1 }} />
        </linearGradient>
      </defs>
      
      <rect 
        x={0} 
        y={0} 
        width={GAME_CONFIG.SCREEN_WIDTH} 
        height={GAME_CONFIG.SCREEN_HEIGHT} 
        fill="url(#skyGradient)" 
      />
      
      {/* Parallax mountain layers */}
      {layers.slice(1).map((layer, index) => {
        const offset = -cameraX * layer.scrollFactor;
        const yPosition = 200 + index * 60;
        
        return (
          <g key={index} transform={`translate(${offset % 400}, 0)`}>
            {/* Repeat pattern across screen */}
            {Array.from({ length: 5 }).map((_, i) => (
              <polygon
                key={i}
                points={`
                  ${i * 400},${GAME_CONFIG.SCREEN_HEIGHT}
                  ${i * 400 + 100},${yPosition}
                  ${i * 400 + 200},${yPosition + 40}
                  ${i * 400 + 300},${yPosition - 20}
                  ${i * 400 + 400},${GAME_CONFIG.SCREEN_HEIGHT}
                `}
                fill={layer.color}
                opacity={0.8 - index * 0.15}
              />
            ))}
          </g>
        );
      })}
      
      {/* Stars (optional decoration) */}
      {Array.from({ length: 30 }).map((_, i) => (
        <circle
          key={`star-${i}`}
          cx={(i * 73) % GAME_CONFIG.SCREEN_WIDTH}
          cy={(i * 37) % (GAME_CONFIG.SCREEN_HEIGHT * 0.4)}
          r={1 + (i % 2)}
          fill="#ffffff"
          opacity={0.3 + (i % 3) * 0.2}
        />
      ))}
    </g>
  );
});

export default Background;
```

### Platform Component
```jsx
// Gameplay/Platform.jsx
import React, { memo } from 'react';
import { GAME_CONFIG } from '../constants';

const Platform = memo(function Platform({ platform, cameraX }) {
  const screenX = platform.x - cameraX;
  
  // Skip if completely off-screen
  if (screenX + platform.width < 0 || screenX > GAME_CONFIG.SCREEN_WIDTH) {
    return null;
  }
  
  const isGround = platform.type === 'ground';
  
  return (
    <g transform={`translate(${screenX}, ${platform.y})`}>
      {/* Main platform body */}
      <rect
        x={0}
        y={0}
        width={platform.width}
        height={platform.height}
        fill={isGround ? '#3d2914' : '#5a4a3a'}
        stroke={isGround ? '#2a1a0a' : '#4a3a2a'}
        strokeWidth={2}
      />
      
      {/* Top grass/surface */}
      <rect
        x={0}
        y={0}
        width={platform.width}
        height={isGround ? 8 : 4}
        fill={isGround ? '#4a7c23' : '#6b5b4b'}
      />
      
      {/* Texture lines */}
      {Array.from({ length: Math.floor(platform.width / 40) }).map((_, i) => (
        <line
          key={i}
          x1={i * 40 + 20}
          y1={isGround ? 10 : 5}
          x2={i * 40 + 20}
          y2={platform.height - 5}
          stroke={isGround ? '#2a1a0a' : '#3a2a1a'}
          strokeWidth={1}
          opacity={0.3}
        />
      ))}
    </g>
  );
});

export default Platform;
```

---

## 9. Level Editor

### Editor Main Component
```jsx
// LevelEditor/Editor.jsx
import React, { useState, useCallback, useRef } from 'react';
import { GAME_CONFIG, GAME_STATES } from '../constants';
import Toolbar from './Toolbar';
import PropertyPanel from './PropertyPanel';
import LevelPreview from './LevelPreview';

const TOOLS = {
  SELECT: 'select',
  PLATFORM: 'platform',
  ENEMY: 'enemy',
  PLAYER_SPAWN: 'player_spawn',
  END_ZONE: 'end_zone',
  ERASE: 'erase',
};

export default function LevelEditor({ dispatch }) {
  const [level, setLevel] = useState({
    width: 3200,
    height: 450,
    platforms: [
      { x: 0, y: 400, width: 3200, height: 50, type: 'ground', id: 'ground_0' },
    ],
    enemies: [],
    playerSpawn: { x: 100, y: 350 },
    endZone: { x: 3100, y: 0, width: 100, height: 450 },
  });
  
  const [selectedTool, setSelectedTool] = useState(TOOLS.SELECT);
  const [selectedObject, setSelectedObject] = useState(null);
  const [cameraX, setCameraX] = useState(0);
  const [isPlacing, setIsPlacing] = useState(false);
  const [placementStart, setPlacementStart] = useState(null);
  const [zoom, setZoom] = useState(0.5);
  
  const editorRef = useRef(null);
  
  // Convert screen coordinates to world coordinates
  const screenToWorld = useCallback((screenX, screenY) => {
    return {
      x: screenX / zoom + cameraX,
      y: screenY / zoom,
    };
  }, [zoom, cameraX]);
  
  // Handle mouse down
  const handleMouseDown = useCallback((e) => {
    const rect = editorRef.current.getBoundingClientRect();
    const worldPos = screenToWorld(e.clientX - rect.left, e.clientY - rect.top);
    
    switch (selectedTool) {
      case TOOLS.PLATFORM:
        setIsPlacing(true);
        setPlacementStart(worldPos);
        break;
        
      case TOOLS.ENEMY:
        // Snap to grid
        const enemyX = Math.round(worldPos.x / 32) * 32;
        const enemyY = Math.round(worldPos.y / 32) * 32;
        setLevel(prev => ({
          ...prev,
          enemies: [...prev.enemies, {
            x: enemyX,
            y: enemyY,
            type: 'goon',
            id: `enemy_${Date.now()}`,
          }],
        }));
        break;
        
      case TOOLS.PLAYER_SPAWN:
        setLevel(prev => ({
          ...prev,
          playerSpawn: { x: worldPos.x, y: worldPos.y },
        }));
        break;
        
      case TOOLS.SELECT:
        // Find clicked object
        const clickedPlatform = level.platforms.find(p => 
          worldPos.x >= p.x && worldPos.x <= p.x + p.width &&
          worldPos.y >= p.y && worldPos.y <= p.y + p.height
        );
        const clickedEnemy = level.enemies.find(e =>
          worldPos.x >= e.x && worldPos.x <= e.x + 28 &&
          worldPos.y >= e.y && worldPos.y <= e.y + 44
        );
        setSelectedObject(clickedPlatform || clickedEnemy || null);
        break;
        
      case TOOLS.ERASE:
        // Remove clicked object
        setLevel(prev => ({
          ...prev,
          platforms: prev.platforms.filter(p => 
            !(worldPos.x >= p.x && worldPos.x <= p.x + p.width &&
              worldPos.y >= p.y && worldPos.y <= p.y + p.height)
          ),
          enemies: prev.enemies.filter(e =>
            !(worldPos.x >= e.x && worldPos.x <= e.x + 28 &&
              worldPos.y >= e.y && worldPos.y <= e.y + 44)
          ),
        }));
        break;
    }
  }, [selectedTool, screenToWorld, level]);
  
  // Handle mouse up (complete platform placement)
  const handleMouseUp = useCallback((e) => {
    if (isPlacing && placementStart && selectedTool === TOOLS.PLATFORM) {
      const rect = editorRef.current.getBoundingClientRect();
      const worldPos = screenToWorld(e.clientX - rect.left, e.clientY - rect.top);
      
      const x = Math.min(placementStart.x, worldPos.x);
      const y = Math.min(placementStart.y, worldPos.y);
      const width = Math.abs(worldPos.x - placementStart.x);
      const height = Math.abs(worldPos.y - placementStart.y);
      
      if (width > 10 && height > 5) {
        setLevel(prev => ({
          ...prev,
          platforms: [...prev.platforms, {
            x: Math.round(x / 16) * 16,
            y: Math.round(y / 16) * 16,
            width: Math.round(width / 16) * 16,
            height: Math.round(height / 16) * 16,
            type: 'platform',
            id: `platform_${Date.now()}`,
          }],
        }));
      }
    }
    
    setIsPlacing(false);
    setPlacementStart(null);
  }, [isPlacing, placementStart, selectedTool, screenToWorld]);
  
  // Export level
  const exportLevel = useCallback(() => {
    const levelData = {
      ...level,
      id: 'custom_' + Date.now(),
      name: 'Custom Level',
    };
    const json = JSON.stringify(levelData, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'custom_level.json';
    a.click();
    URL.revokeObjectURL(url);
  }, [level]);
  
  // Import level
  const importLevel = useCallback((e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const data = JSON.parse(event.target.result);
          setLevel(data);
        } catch (err) {
          alert('Invalid level file');
        }
      };
      reader.readAsText(file);
    }
  }, []);
  
  // Test play level
  const testPlay = useCallback(() => {
    dispatch({ type: 'LOAD_LEVEL', payload: level, enemies: level.enemies });
    dispatch({ type: 'SET_GAME_STATE', payload: GAME_STATES.PLAYING });
  }, [level, dispatch]);
  
  return (
    <div className="level-editor" style={{
      display: 'flex',
      flexDirection: 'column',
      width: '100%',
      height: '100%',
      backgroundColor: '#1a1a2e',
    }}>
      {/* Toolbar */}
      <Toolbar 
        selectedTool={selectedTool}
        onToolSelect={setSelectedTool}
        onExport={exportLevel}
        onImport={importLevel}
        onTestPlay={testPlay}
        onExit={() => dispatch({ type: 'SET_GAME_STATE', payload: GAME_STATES.MENU })}
        zoom={zoom}
        onZoomChange={setZoom}
      />
      
      <div style={{ display: 'flex', flex: 1 }}>
        {/* Canvas area */}
        <div 
          ref={editorRef}
          className="editor-canvas"
          style={{
            flex: 1,
            overflow: 'hidden',
            cursor: selectedTool === TOOLS.PLATFORM ? 'crosshair' : 'default',
          }}
          onMouseDown={handleMouseDown}
          onMouseUp={handleMouseUp}
          onWheel={(e) => {
            // Scroll to pan
            setCameraX(prev => Math.max(0, Math.min(
              prev + e.deltaX + e.deltaY,
              level.width - GAME_CONFIG.SCREEN_WIDTH
            )));
          }}
        >
          <LevelPreview 
            level={level}
            cameraX={cameraX}
            zoom={zoom}
            selectedObject={selectedObject}
            isPlacing={isPlacing}
            placementStart={placementStart}
          />
        </div>
        
        {/* Property panel */}
        <PropertyPanel 
          selectedObject={selectedObject}
          onUpdate={(updates) => {
            if (selectedObject) {
              setLevel(prev => ({
                ...prev,
                platforms: prev.platforms.map(p => 
                  p.id === selectedObject.id ? { ...p, ...updates } : p
                ),
                enemies: prev.enemies.map(e =>
                  e.id === selectedObject.id ? { ...e, ...updates } : e
                ),
              }));
            }
          }}
          levelSettings={{ width: level.width, height: level.height }}
          onLevelSettingsChange={(settings) => setLevel(prev => ({ ...prev, ...settings }))}
        />
      </div>
    </div>
  );
}
```

### Toolbar Component
```jsx
// LevelEditor/Toolbar.jsx
import React from 'react';

export default function Toolbar({ 
  selectedTool, 
  onToolSelect, 
  onExport, 
  onImport,
  onTestPlay,
  onExit,
  zoom,
  onZoomChange,
}) {
  const tools = [
    { id: 'select', label: '↖ Select', icon: '↖' },
    { id: 'platform', label: '▢ Platform', icon: '▢' },
    { id: 'enemy', label: '☠ Enemy', icon: '☠' },
    { id: 'player_spawn', label: '★ Spawn', icon: '★' },
    { id: 'erase', label: '✕ Erase', icon: '✕' },
  ];
  
  return (
    <div style={{
      display: 'flex',
      gap: '8px',
      padding: '8px',
      backgroundColor: '#16213e',
      borderBottom: '2px solid #1f4068',
      alignItems: 'center',
    }}>
      {/* Tools */}
      {tools.map(tool => (
        <button
          key={tool.id}
          onClick={() => onToolSelect(tool.id)}
          style={{
            padding: '8px 12px',
            backgroundColor: selectedTool === tool.id ? '#e94560' : '#1f4068',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: 'pointer',
            fontFamily: 'monospace',
            fontSize: '14px',
          }}
        >
          {tool.label}
        </button>
      ))}
      
      <div style={{ flex: 1 }} />
      
      {/* Zoom */}
      <span style={{ color: 'white', marginRight: '8px' }}>Zoom:</span>
      <input
        type="range"
        min="0.25"
        max="1"
        step="0.05"
        value={zoom}
        onChange={(e) => onZoomChange(parseFloat(e.target.value))}
        style={{ width: '80px' }}
      />
      
      {/* Actions */}
      <button
        onClick={onTestPlay}
        style={{
          padding: '8px 16px',
          backgroundColor: '#4CAF50',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer',
          fontWeight: 'bold',
        }}
      >
        ▶ Test Play
      </button>
      
      <button
        onClick={onExport}
        style={{
          padding: '8px 16px',
          backgroundColor: '#2196F3',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer',
        }}
      >
        💾 Export
      </button>
      
      <label style={{
        padding: '8px 16px',
        backgroundColor: '#9C27B0',
        color: 'white',
        borderRadius: '4px',
        cursor: 'pointer',
      }}>
        📂 Import
        <input
          type="file"
          accept=".json"
          onChange={onImport}
          style={{ display: 'none' }}
        />
      </label>
      
      <button
        onClick={onExit}
        style={{
          padding: '8px 16px',
          backgroundColor: '#f44336',
          color: 'white',
          border: 'none',
          borderRadius: '4px',
          cursor: 'pointer',
        }}
      >
        ✕ Exit
      </button>
    </div>
  );
}
```

### Property Panel
```jsx
// LevelEditor/PropertyPanel.jsx
import React from 'react';

export default function PropertyPanel({ 
  selectedObject, 
  onUpdate, 
  levelSettings,
  onLevelSettingsChange,
}) {
  return (
    <div style={{
      width: '200px',
      backgroundColor: '#16213e',
      borderLeft: '2px solid #1f4068',
      padding: '12px',
      color: 'white',
      fontFamily: 'monospace',
      fontSize: '12px',
    }}>
      <h3 style={{ margin: '0 0 12px 0', color: '#e94560' }}>Properties</h3>
      
      {/* Level Settings */}
      <div style={{ marginBottom: '16px' }}>
        <h4 style={{ margin: '0 0 8px 0', color: '#4fc3f7' }}>Level</h4>
        <label style={{ display: 'block', marginBottom: '4px' }}>
          Width:
          <input
            type="number"
            value={levelSettings.width}
            onChange={(e) => onLevelSettingsChange({ width: parseInt(e.target.value) })}
            style={{ width: '100%', marginTop: '2px' }}
          />
        </label>
        <label style={{ display: 'block' }}>
          Height:
          <input
            type="number"
            value={levelSettings.height}
            onChange={(e) => onLevelSettingsChange({ height: parseInt(e.target.value) })}
            style={{ width: '100%', marginTop: '2px' }}
          />
        </label>
      </div>
      
      {/* Selected Object Properties */}
      {selectedObject && (
        <div>
          <h4 style={{ margin: '0 0 8px 0', color: '#4fc3f7' }}>
            {selectedObject.type || 'Object'}
          </h4>
          
          <label style={{ display: 'block', marginBottom: '4px' }}>
            X:
            <input
              type="number"
              value={selectedObject.x}
              onChange={(e) => onUpdate({ x: parseInt(e.target.value) })}
              style={{ width: '100%', marginTop: '2px' }}
            />
          </label>
          
          <label style={{ display: 'block', marginBottom: '4px' }}>
            Y:
            <input
              type="number"
              value={selectedObject.y}
              onChange={(e) => onUpdate({ y: parseInt(e.target.value) })}
              style={{ width: '100%', marginTop: '2px' }}
            />
          </label>
          
          {selectedObject.width !== undefined && (
            <label style={{ display: 'block', marginBottom: '4px' }}>
              Width:
              <input
                type="number"
                value={selectedObject.width}
                onChange={(e) => onUpdate({ width: parseInt(e.target.value) })}
                style={{ width: '100%', marginTop: '2px' }}
              />
            </label>
          )}
          
          {selectedObject.height !== undefined && (
            <label style={{ display: 'block', marginBottom: '4px' }}>
              Height:
              <input
                type="number"
                value={selectedObject.height}
                onChange={(e) => onUpdate({ height: parseInt(e.target.value) })}
                style={{ width: '100%', marginTop: '2px' }}
              />
            </label>
          )}
        </div>
      )}
      
      {!selectedObject && (
        <p style={{ color: '#888', fontStyle: 'italic' }}>
          Click an object to edit its properties
        </p>
      )}
    </div>
  );
}
```

---

## 10. Menu System

### Main Menu
```jsx
// Menu/MainMenu.jsx
import React, { useState } from 'react';
import { GAME_STATES, GAME_CONFIG } from '../constants';

export default function MainMenu({ onStateChange }) {
  const [showOptions, setShowOptions] = useState(false);
  
  const menuItems = [
    { label: 'START GAME', action: () => onStateChange(GAME_STATES.PLAYING) },
    { label: 'LEVEL EDITOR', action: () => onStateChange(GAME_STATES.LEVEL_EDITOR) },
    { label: 'OPTIONS', action: () => setShowOptions(true) },
  ];
  
  return (
    <svg 
      width={GAME_CONFIG.SCREEN_WIDTH} 
      height={GAME_CONFIG.SCREEN_HEIGHT}
      style={{ display: 'block' }}
    >
      {/* Background */}
      <defs>
        <linearGradient id="menuBg" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" style={{ stopColor: '#0f0c29' }} />
          <stop offset="50%" style={{ stopColor: '#302b63' }} />
          <stop offset="100%" style={{ stopColor: '#24243e' }} />
        </linearGradient>
        
        {/* Title glow effect */}
        <filter id="glow">
          <feGaussianBlur stdDeviation="3" result="coloredBlur" />
          <feMerge>
            <feMergeNode in="coloredBlur" />
            <feMergeNode in="SourceGraphic" />
          </feMerge>
        </filter>
      </defs>
      
      <rect 
        width={GAME_CONFIG.SCREEN_WIDTH} 
        height={GAME_CONFIG.SCREEN_HEIGHT} 
        fill="url(#menuBg)" 
      />
      
      {/* Animated scan lines */}
      {Array.from({ length: 20 }).map((_, i) => (
        <line
          key={i}
          x1={0}
          y1={i * 25}
          x2={GAME_CONFIG.SCREEN_WIDTH}
          y2={i * 25}
          stroke="#ffffff"
          strokeWidth={1}
          opacity={0.03}
        />
      ))}
      
      {/* Title */}
      <g transform={`translate(${GAME_CONFIG.SCREEN_WIDTH / 2}, 80)`}>
        <text
          textAnchor="middle"
          fill="#e94560"
          fontSize="48"
          fontWeight="bold"
          fontFamily="Impact, sans-serif"
          filter="url(#glow)"
          style={{ textTransform: 'uppercase' }}
        >
          METAL SLUG
        </text>
        <text
          textAnchor="middle"
          y={40}
          fill="#4fc3f7"
          fontSize="24"
          fontFamily="monospace"
        >
          REACT EDITION
        </text>
      </g>
      
      {/* Player character decoration */}
      <g transform={`translate(100, 250)`}>
        <rect x={8} y={0} width={16} height={12} fill="#f4a460" /> {/* Head */}
        <rect x={6} y={12} width={20} height={16} fill="#228b22" /> {/* Body */}
        <rect x={24} y={14} width={16} height={5} fill="#f4a460" /> {/* Arm */}
        <rect x={38} y={12} width={8} height={4} fill="#333" /> {/* Gun */}
        <rect x={6} y={28} width={8} height={12} fill="#2f4f4f" /> {/* Legs */}
        <rect x={18} y={28} width={8} height={12} fill="#2f4f4f" />
      </g>
      
      {/* Enemy decoration */}
      <g transform={`translate(650, 250) scale(-1, 1)`}>
        <ellipse cx={16} cy={5} rx={14} ry={5} fill="#1e3a5f" />
        <rect x={8} y={4} width={16} height={10} fill="#d4a574" />
        <rect x={5} y={14} width={22} height={16} fill="#8b4513" />
        <rect x={6} y={30} width={8} height={12} fill="#5d4e37" />
        <rect x={18} y={30} width={8} height={12} fill="#5d4e37" />
      </g>
      
      {/* Menu items */}
      <g transform={`translate(${GAME_CONFIG.SCREEN_WIDTH / 2}, 200)`}>
        {menuItems.map((item, index) => (
          <g
            key={item.label}
            transform={`translate(0, ${index * 60})`}
            style={{ cursor: 'pointer' }}
            onClick={item.action}
          >
            {/* Button background */}
            <rect
              x={-120}
              y={0}
              width={240}
              height={45}
              fill="#1f4068"
              stroke="#e94560"
              strokeWidth={2}
              rx={4}
            />
            
            {/* Button text */}
            <text
              textAnchor="middle"
              y={30}
              fill="white"
              fontSize="20"
              fontWeight="bold"
              fontFamily="monospace"
            >
              {item.label}
            </text>
            
            {/* Hover effect overlay */}
            <rect
              x={-120}
              y={0}
              width={240}
              height={45}
              fill="white"
              opacity={0}
              rx={4}
              style={{ transition: 'opacity 0.2s' }}
              onMouseEnter={(e) => e.target.style.opacity = 0.1}
              onMouseLeave={(e) => e.target.style.opacity = 0}
            />
          </g>
        ))}
      </g>
      
      {/* Footer */}
      <text
        x={GAME_CONFIG.SCREEN_WIDTH / 2}
        y={GAME_CONFIG.SCREEN_HEIGHT - 20}
        textAnchor="middle"
        fill="#666"
        fontSize="12"
        fontFamily="monospace"
      >
        Press F3 for Debug Mode | Arrow Keys to Move | Z to Shoot | X to Kick
      </text>
      
      {/* Options overlay */}
      {showOptions && (
        <OptionsOverlay onClose={() => setShowOptions(false)} />
      )}
    </svg>
  );
}

function OptionsOverlay({ onClose }) {
  return (
    <g>
      {/* Backdrop */}
      <rect 
        width={GAME_CONFIG.SCREEN_WIDTH} 
        height={GAME_CONFIG.SCREEN_HEIGHT} 
        fill="black" 
        opacity={0.7}
        onClick={onClose}
      />
      
      {/* Options panel */}
      <g transform={`translate(${GAME_CONFIG.SCREEN_WIDTH / 2 - 150}, 100)`}>
        <rect
          x={0}
          y={0}
          width={300}
          height={250}
          fill="#1f4068"
          stroke="#e94560"
          strokeWidth={3}
          rx={8}
        />
        
        <text x={150} y={40} textAnchor="middle" fill="white" fontSize="24" fontWeight="bold">
          OPTIONS
        </text>
        
        {/* Volume sliders would go here */}
        <text x={30} y={80} fill="#ccc" fontSize="14">Sound Volume</text>
        <rect x={30} y={90} width={240} height={10} fill="#333" rx={5} />
        <rect x={30} y={90} width={170} height={10} fill="#4fc3f7" rx={5} />
        
        <text x={30} y={130} fill="#ccc" fontSize="14">Music Volume</text>
        <rect x={30} y={140} width={240} height={10} fill="#333" rx={5} />
        <rect x={30} y={140} width={168} height={10} fill="#4fc3f7" rx={5} />
        
        {/* Close button */}
        <g 
          transform="translate(100, 180)"
          style={{ cursor: 'pointer' }}
          onClick={onClose}
        >
          <rect x={0} y={0} width={100} height={40} fill="#e94560" rx={4} />
          <text x={50} y={27} textAnchor="middle" fill="white" fontSize="16" fontWeight="bold">
            CLOSE
          </text>
        </g>
      </g>
    </g>
  );
}
```

### HUD Component
```jsx
// UI/HUD.jsx
import React, { memo } from 'react';
import { GAME_CONFIG, PLAYER_STATES } from '../constants';

const HUD = memo(function HUD({ player, score, level, debug }) {
  return (
    <g className="hud">
      {/* Score */}
      <text
        x={20}
        y={30}
        fill="#ffcc00"
        fontSize="24"
        fontWeight="bold"
        fontFamily="monospace"
      >
        SCORE: {score.toString().padStart(8, '0')}
      </text>
      
      {/* Level indicator */}
      <text
        x={GAME_CONFIG.SCREEN_WIDTH - 20}
        y={30}
        textAnchor="end"
        fill="#4fc3f7"
        fontSize="18"
        fontFamily="monospace"
      >
        LEVEL {level}
      </text>
      
      {/* Player status */}
      {player.state === PLAYER_STATES.CAPTURED && (
        <g transform={`translate(${GAME_CONFIG.SCREEN_WIDTH / 2}, ${GAME_CONFIG.SCREEN_HEIGHT / 2})`}>
          <text
            textAnchor="middle"
            fill="#e94560"
            fontSize="36"
            fontWeight="bold"
            fontFamily="Impact, sans-serif"
          >
            CAPTURED!
          </text>
          <text
            textAnchor="middle"
            y={40}
            fill="#ccc"
            fontSize="16"
            fontFamily="monospace"
          >
            Press SPACE to continue
          </text>
        </g>
      )}
      
      {/* Weapon indicator */}
      <g transform={`translate(20, ${GAME_CONFIG.SCREEN_HEIGHT - 50})`}>
        <rect x={0} y={0} width={80} height={30} fill="#1f4068" stroke="#4fc3f7" strokeWidth={1} rx={4} />
        <text x={40} y={21} textAnchor="middle" fill="white" fontSize="14" fontFamily="monospace">
          {player.weapon.toUpperCase()}
        </text>
      </g>
      
      {/* Ammo (if special weapon) */}
      {player.specialAmmo > 0 && (
        <g transform={`translate(110, ${GAME_CONFIG.SCREEN_HEIGHT - 50})`}>
          <text fill="#ffcc00" fontSize="14" fontFamily="monospace">
            AMMO: {player.specialAmmo}
          </text>
        </g>
      )}
      
      {/* Debug info */}
      {debug?.enabled && (
        <g transform={`translate(20, 60)`}>
          <text fill="#0f0" fontSize="10" fontFamily="monospace">
            X: {Math.round(player.x)} Y: {Math.round(player.y)}
          </text>
          <text y={12} fill="#0f0" fontSize="10" fontFamily="monospace">
            VX: {player.vx.toFixed(2)} VY: {player.vy.toFixed(2)}
          </text>
          <text y={24} fill="#0f0" fontSize="10" fontFamily="monospace">
            State: {player.state}
          </text>
          <text y={36} fill="#0f0" fontSize="10" fontFamily="monospace">
            Grounded: {player.isGrounded ? 'Yes' : 'No'}
          </text>
        </g>
      )}
    </g>
  );
});

export default HUD;
```

---

## 11. Audio System

### Audio Manager
```javascript
// systems/audio.js
class AudioManager {
  constructor() {
    this.sounds = {};
    this.musicVolume = 0.7;
    this.sfxVolume = 0.8;
    this.currentMusic = null;
  }
  
  // Preload sounds (placeholder - would load actual audio files)
  preload() {
    // In a real implementation, you'd load actual audio files
    // For now, we'll use the Web Audio API to generate simple sounds
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    this.sounds = {
      shoot: this.createBeepSound(800, 0.1),
      enemyShoot: this.createBeepSound(400, 0.1),
      hit: this.createNoiseSound(0.15),
      kick: this.createBeepSound(200, 0.1),
      jump: this.createBeepSound(600, 0.1),
      enemyDeath: this.createNoiseSound(0.3),
      pickup: this.createBeepSound(1000, 0.15),
    };
  }
  
  createBeepSound(frequency, duration) {
    return () => {
      if (!this.audioContext) return;
      
      const oscillator = this.audioContext.createOscillator();
      const gainNode = this.audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(this.audioContext.destination);
      
      oscillator.frequency.value = frequency;
      oscillator.type = 'square';
      
      gainNode.gain.setValueAtTime(this.sfxVolume * 0.3, this.audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, this.audioContext.currentTime + duration);
      
      oscillator.start(this.audioContext.currentTime);
      oscillator.stop(this.audioContext.currentTime + duration);
    };
  }
  
  createNoiseSound(duration) {
    return () => {
      if (!this.audioContext) return;
      
      const bufferSize = this.audioContext.sampleRate * duration;
      const buffer = this.audioContext.createBuffer(1, bufferSize, this.audioContext.sampleRate);
      const data = buffer.getChannelData(0);
      
      for (let i = 0; i < bufferSize; i++) {
        data[i] = Math.random() * 2 - 1;
      }
      
      const noise = this.audioContext.createBufferSource();
      const gainNode = this.audioContext.createGain();
      
      noise.buffer = buffer;
      noise.connect(gainNode);
      gainNode.connect(this.audioContext.destination);
      
      gainNode.gain.setValueAtTime(this.sfxVolume * 0.2, this.audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, this.audioContext.currentTime + duration);
      
      noise.start();
    };
  }
  
  play(soundName) {
    if (this.sounds[soundName]) {
      this.sounds[soundName]();
    }
  }
  
  setMusicVolume(volume) {
    this.musicVolume = volume;
  }
  
  setSFXVolume(volume) {
    this.sfxVolume = volume;
  }
}

export const audioManager = new AudioManager();
```

### Audio Hook
```javascript
// hooks/useAudio.js
import { useEffect, useCallback, useRef } from 'react';
import { audioManager } from '../systems/audio';

export function useAudio() {
  const initialized = useRef(false);
  
  useEffect(() => {
    if (!initialized.current) {
      // Initialize audio on first user interaction
      const initAudio = () => {
        audioManager.preload();
        initialized.current = true;
        window.removeEventListener('click', initAudio);
        window.removeEventListener('keydown', initAudio);
      };
      
      window.addEventListener('click', initAudio);
      window.addEventListener('keydown', initAudio);
      
      return () => {
        window.removeEventListener('click', initAudio);
        window.removeEventListener('keydown', initAudio);
      };
    }
  }, []);
  
  const playSound = useCallback((soundName) => {
    audioManager.play(soundName);
  }, []);
  
  return { playSound };
}
```

---

## 12. Implementation Phases

### Phase 1: Core Foundation (Week 1)
1. ✅ Set up React project structure
2. ✅ Implement constants and game state
3. ✅ Create game loop hook
4. ✅ Create input handling hook
5. ✅ Build main Game component with state routing

### Phase 2: Player System (Week 1-2)
1. ✅ Create Player component with SVG sprites
2. ✅ Implement all player states and animations
3. ✅ Build player physics (gravity, velocity)
4. ✅ Add platform collision detection
5. ✅ Implement all player actions (walk, jump, roll, shoot, kick)

### Phase 3: Enemy System (Week 2)
1. ✅ Create Enemy component with SVG sprites
2. ✅ Implement enemy AI state machine
3. ✅ Add enemy behaviors (patrol, chase, attack, shoot)
4. ✅ Create enemy death animations
5. ✅ Implement enemy spawning system

### Phase 4: Combat System (Week 2-3)
1. ✅ Create Bullet component
2. ✅ Implement collision detection system
3. ✅ Add player kick attack
4. ✅ Handle enemy attacks
5. ✅ Implement player capture state

### Phase 5: Level System (Week 3)
1. ✅ Create level data structure
2. ✅ Build platform rendering
3. ✅ Implement parallax background
4. ✅ Add camera following with smooth scrolling
5. ✅ Create level loading system

### Phase 6: Level Editor (Week 3-4)
1. ✅ Build editor UI framework
2. ✅ Implement object placement tools
3. ✅ Add property editing panel
4. ✅ Create level export/import
5. ✅ Add test play functionality

### Phase 7: UI & Polish (Week 4)
1. ✅ Create main menu
2. ✅ Build HUD system
3. ✅ Add audio system
4. ✅ Implement game over screen
5. ✅ Add debug overlay

### Phase 8: Testing & Refinement (Week 4+)
1. Test all game mechanics
2. Balance enemy difficulty
3. Polish animations
4. Optimize performance
5. Prepare for PNG sprite replacement

---

## Sprite Replacement Guide

When ready to replace SVG placeholders with PNG sprites:

### Directory Structure
```
/public/sprites/
├── player/
│   ├── idle_1.png
│   ├── idle_2.png
│   ├── idle_3.png
│   ├── walk_1.png
│   ├── walk_2.png
│   ├── walk_3.png
│   ├── jump_1.png
│   ├── jump_2.png
│   ├── jump_3.png
│   ├── roll_1.png
│   ├── roll_2.png
│   ├── roll_3.png
│   ├── shoot_1.png
│   ├── shoot_2.png
│   ├── shoot_3.png
│   ├── kick_1.png
│   ├── kick_2.png
│   ├── kick_3.png
│   ├── captured_1.png
│   ├── captured_2.png
│   └── captured_3.png
└── enemy/
    ├── goon/
    │   ├── idle_1.png
    │   ├── ...
```

### Sprite Component Update
```jsx
// Replace SVG sprite with image
export default function PlayerSprite({ state, direction, frame, width, height }) {
  const imagePath = `/sprites/player/${state}_${frame + 1}.png`;
  
  return (
    <image
      href={imagePath}
      width={width}
      height={height}
      transform={direction === DIRECTIONS.LEFT ? `scale(-1, 1) translate(-${width}, 0)` : ''}
      style={{ imageRendering: 'pixelated' }}
    />
  );
}
```

---

## Quick Start

To begin implementation, create a single React file (for artifact) that includes all core systems. The full implementation should follow the modular structure above for maintainability.

**Key Files to Create First:**
1. `constants.js` - All game configuration
2. `Game.jsx` - Main container
3. `useGameLoop.js` - Animation frame loop
4. `useInput.js` - Keyboard handling
5. `Player.jsx` + `PlayerSprite.jsx` - Player rendering
6. `GameCanvas.jsx` - Main gameplay area

This plan provides a complete roadmap for building a Metal Slug-inspired side-scroller with all the requested features. Each section includes working code that can be directly implemented.
