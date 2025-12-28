# Mecha Neural Network Learning System

## Overview

This document specifies the integration of real neural networks to replace the existing behaviour tree AI system. The new system enables mechas to **learn** through simulated battles, with trained networks exportable as portable files for tournament entry.

---

## Core Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                      MECHA LEARNING SYSTEM                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │   Training   │───▶│    Neural    │───▶│  Export/Import   │  │
│  │   Manager    │    │   Network    │    │    (.mecha-ai)   │  │
│  └──────────────┘    └──────────────┘    └──────────────────┘  │
│         │                   │                     │             │
│         ▼                   ▼                     ▼             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │  Simulation  │    │   Decision   │    │    Tournament    │  │
│  │    Engine    │    │    Output    │    │      Bundle      │  │
│  └──────────────┘    └──────────────┘    └──────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Neural Network Specification

### Library

Use **TensorFlow.js** for browser-native training and inference.

```bash
npm install @tensorflow/tfjs
```

### Network Architecture

The network operates as a **policy network** that takes game state as input and outputs action probabilities.

```
INPUT LAYER
├── Mech State Inputs (read from existing mech config schema)
├── Enemy State Inputs (mirror of mech state)
├── Arena State Inputs (distance, terrain, boundaries)
└── Temporal Inputs (cooldowns, match time remaining)

HIDDEN LAYERS
├── Dense Layer 1: 128 neurons, ReLU activation
├── Dense Layer 2: 64 neurons, ReLU activation
└── Dense Layer 3: 32 neurons, ReLU activation

OUTPUT LAYER
└── Action Probabilities (softmax)
    ├── Movement actions
    ├── Attack actions  
    ├── Defensive actions
    └── Special ability actions
```

### Input/Output Tensor Structure

```typescript
// NOTE: Exact fields to be populated from existing mech stat schema
// Reference: [YOUR_CODEBASE]/types/mech.ts or equivalent

interface NetworkInputs {
  // Self state - derive fields from existing MechConfig/MechStats types
  selfState: number[];      // [...mech parameters from codebase]
  
  // Enemy state - same structure as selfState
  enemyState: number[];
  
  // Arena context
  arenaState: {
    distanceToEnemy: number;      // normalized 0-1
    angleToEnemy: number;         // normalized -1 to 1
    distanceToBoundary: number;   // normalized 0-1
    // ... additional arena params from codebase
  };
  
  // Temporal context
  temporalState: {
    matchTimeRemaining: number;   // normalized 0-1
    // cooldowns per ability - derive from codebase
  };
}

// Output actions - derive from existing action/ability system
interface NetworkOutputs {
  // Reference: existing action enum/types in codebase
  actionProbabilities: number[];  // softmax over available actions
}
```

---

## Training System

### Training Manager

```typescript
// src/ai/TrainingManager.ts

import * as tf from '@tensorflow/tfjs';

interface TrainingConfig {
  simulationsPerBatch: number;    // recommend: 100
  totalBatches: number;           // recommend: 100 (10,000 total sims)
  learningRate: number;           // recommend: 0.001
  discountFactor: number;         // gamma for rewards, recommend: 0.99
  explorationRate: number;        // epsilon for exploration, starts at 1.0
  explorationDecay: number;       // recommend: 0.995
  minExplorationRate: number;     // recommend: 0.01
}

class TrainingManager {
  private model: tf.LayersModel;
  private config: TrainingConfig;
  private trainingHistory: TrainingMetrics[];

  constructor(mechConfig: MechConfig, config: TrainingConfig) {
    this.model = this.buildModel(mechConfig);
    this.config = config;
  }

  private buildModel(mechConfig: MechConfig): tf.LayersModel {
    // INPUT SIZE: Calculate from mechConfig
    // Reference codebase for exact parameter count
    const inputSize = this.calculateInputSize(mechConfig);
    
    // OUTPUT SIZE: Number of possible actions
    // Reference codebase action/ability types
    const outputSize = this.calculateOutputSize(mechConfig);

    const model = tf.sequential({
      layers: [
        tf.layers.dense({ inputShape: [inputSize], units: 128, activation: 'relu' }),
        tf.layers.dense({ units: 64, activation: 'relu' }),
        tf.layers.dense({ units: 32, activation: 'relu' }),
        tf.layers.dense({ units: outputSize, activation: 'softmax' })
      ]
    });

    model.compile({
      optimizer: tf.train.adam(this.config.learningRate),
      loss: 'categoricalCrossentropy',
      metrics: ['accuracy']
    });

    return model;
  }

  async runTrainingSession(
    opponentPool: MechConfig[],
    onProgress?: (progress: TrainingProgress) => void
  ): Promise<TrainedNetwork> {
    // Implementation uses existing simulation engine
    // Reference: existing battle simulation code in codebase
  }
}
```

### Reward Function

```typescript
// src/ai/RewardCalculator.ts

interface BattleOutcome {
  // Reference existing battle result types from codebase
  winner: 'self' | 'enemy' | 'draw';
  finalHealthSelf: number;
  finalHealthEnemy: number;
  damageDealt: number;
  damageTaken: number;
  matchDuration: number;
  // ... additional metrics from existing system
}

class RewardCalculator {
  calculate(outcome: BattleOutcome, action: Action, gameState: GameState): number {
    let reward = 0;

    // Terminal rewards
    if (outcome.winner === 'self') reward += 1.0;
    else if (outcome.winner === 'enemy') reward -= 1.0;

    // Shaping rewards (tune based on game balance)
    reward += outcome.damageDealt * 0.1;
    reward -= outcome.damageTaken * 0.1;
    
    // Efficiency bonus
    reward += (outcome.finalHealthSelf / maxHealth) * 0.2;

    // Time pressure (optional - encourage decisive play)
    if (outcome.winner === 'self') {
      reward += (1 - outcome.matchDuration / maxDuration) * 0.1;
    }

    return reward;
  }
}
```

### Simulation Integration

```typescript
// src/ai/SimulationRunner.ts

class SimulationRunner {
  constructor(
    private simulationEngine: ExistingSimulationEngine  // Reference your existing sim
  ) {}

  async runSimulation(
    selfMech: MechConfig,
    selfNetwork: tf.LayersModel,
    opponentMech: MechConfig,
    opponentAI: tf.LayersModel | LegacyPersonaAI  // Support both
  ): Promise<SimulationResult> {
    
    // Use existing deterministic simulation
    // Replace behaviour tree decision points with network inference
    
    const gameState = this.simulationEngine.initialize(selfMech, opponentMech);
    const experienceBuffer: Experience[] = [];

    while (!gameState.isTerminal) {
      // Get state tensor
      const stateTensor = this.gameStateToTensor(gameState);
      
      // Network inference
      const actionProbs = selfNetwork.predict(stateTensor) as tf.Tensor;
      const action = this.sampleAction(actionProbs, explorationRate);
      
      // Execute in simulation
      // Reference: existing simulation step function
      const nextState = this.simulationEngine.step(gameState, action);
      
      // Store experience
      experienceBuffer.push({
        state: stateTensor,
        action: action,
        reward: this.calculateStepReward(gameState, nextState, action),
        nextState: this.gameStateToTensor(nextState),
        done: nextState.isTerminal
      });

      gameState = nextState;
    }

    return { outcome: gameState.outcome, experiences: experienceBuffer };
  }
}
```

---

## Network Export/Import

### File Format: `.mecha-ai`

The exported file is a JSON bundle containing the network weights and metadata.

```typescript
// src/ai/NetworkSerializer.ts

interface MechaAIFile {
  version: string;              // Schema version for compatibility
  mechConfigHash: string;       // Hash of mech config for validation
  network: {
    architecture: LayerConfig[];
    weights: SerializedWeights;
  };
  training: {
    totalSimulations: number;
    finalExplorationRate: number;
    averageReward: number;
    winRate: number;
    trainingDate: string;
  };
  metadata: {
    mechName: string;
    ownerDisplayName: string;
    // ... additional metadata
  };
}

class NetworkSerializer {
  async export(
    model: tf.LayersModel,
    mechConfig: MechConfig,
    trainingMetrics: TrainingMetrics
  ): Promise<Blob> {
    
    const weights = await this.serializeWeights(model);
    
    const mechaAI: MechaAIFile = {
      version: '1.0.0',
      mechConfigHash: this.hashConfig(mechConfig),
      network: {
        architecture: this.extractArchitecture(model),
        weights: weights
      },
      training: {
        totalSimulations: trainingMetrics.totalSimulations,
        finalExplorationRate: trainingMetrics.explorationRate,
        averageReward: trainingMetrics.averageReward,
        winRate: trainingMetrics.winRate,
        trainingDate: new Date().toISOString()
      },
      metadata: {
        mechName: mechConfig.name,
        ownerDisplayName: mechConfig.ownerDisplayName
      }
    };

    return new Blob(
      [JSON.stringify(mechaAI)],
      { type: 'application/json' }
    );
  }

  async import(file: Blob, mechConfig: MechConfig): Promise<tf.LayersModel> {
    const mechaAI: MechaAIFile = JSON.parse(await file.text());
    
    // Validate compatibility
    if (mechaAI.mechConfigHash !== this.hashConfig(mechConfig)) {
      throw new Error('Network incompatible with current mech configuration');
    }

    const model = this.rebuildModel(mechaAI.network.architecture);
    await this.loadWeights(model, mechaAI.network.weights);
    
    return model;
  }
}
```

### Tournament Bundle

```typescript
// src/tournament/TournamentBundle.ts

interface TournamentEntry {
  mechConfig: MechConfig;       // Existing mech configuration
  trainedAI: MechaAIFile;       // Neural network file
  entryTimestamp: string;
  signature?: string;           // Optional: integrity verification
}

class TournamentBundler {
  async createEntry(
    mechConfig: MechConfig,
    trainedNetwork: tf.LayersModel,
    trainingMetrics: TrainingMetrics
  ): Promise<Blob> {
    
    const serializer = new NetworkSerializer();
    const aiFile = await serializer.export(trainedNetwork, mechConfig, trainingMetrics);
    
    const entry: TournamentEntry = {
      mechConfig: mechConfig,
      trainedAI: JSON.parse(await aiFile.text()),
      entryTimestamp: new Date().toISOString()
    };

    return new Blob(
      [JSON.stringify(entry)],
      { type: 'application/json' }
    );
  }
}
```

---

## React Integration

### Training UI Component

```tsx
// src/components/MechaTraining.tsx

interface TrainingUIProps {
  mechConfig: MechConfig;
  onTrainingComplete: (network: tf.LayersModel, metrics: TrainingMetrics) => void;
}

const MechaTraining: React.FC<TrainingUIProps> = ({ mechConfig, onTrainingComplete }) => {
  const [progress, setProgress] = useState<TrainingProgress | null>(null);
  const [isTraining, setIsTraining] = useState(false);
  const trainingManager = useRef<TrainingManager | null>(null);

  const startTraining = async () => {
    setIsTraining(true);
    
    trainingManager.current = new TrainingManager(mechConfig, {
      simulationsPerBatch: 100,
      totalBatches: 100,
      learningRate: 0.001,
      discountFactor: 0.99,
      explorationRate: 1.0,
      explorationDecay: 0.995,
      minExplorationRate: 0.01
    });

    // Fetch opponent pool - reference existing opponent/matchmaking system
    const opponents = await fetchOpponentPool();

    const result = await trainingManager.current.runTrainingSession(
      opponents,
      (p) => setProgress(p)
    );

    setIsTraining(false);
    onTrainingComplete(result.model, result.metrics);
  };

  return (
    <div className="mecha-training">
      <h2>Train {mechConfig.name}</h2>
      
      {!isTraining && (
        <button onClick={startTraining}>
          Begin Training (10,000 simulations)
        </button>
      )}

      {isTraining && progress && (
        <TrainingProgressDisplay progress={progress} />
      )}
    </div>
  );
};
```

### Progress Display

```tsx
// src/components/TrainingProgressDisplay.tsx

interface TrainingProgress {
  currentBatch: number;
  totalBatches: number;
  simulationsComplete: number;
  currentWinRate: number;
  averageReward: number;
  explorationRate: number;
  recentOpponents: string[];
}

const TrainingProgressDisplay: React.FC<{ progress: TrainingProgress }> = ({ progress }) => {
  const percentComplete = (progress.currentBatch / progress.totalBatches) * 100;
  
  return (
    <div className="training-progress">
      <div className="progress-bar">
        <div style={{ width: `${percentComplete}%` }} />
      </div>
      
      <div className="metrics">
        <div>Simulations: {progress.simulationsComplete.toLocaleString()}</div>
        <div>Win Rate: {(progress.currentWinRate * 100).toFixed(1)}%</div>
        <div>Avg Reward: {progress.averageReward.toFixed(3)}</div>
        <div>Exploration: {(progress.explorationRate * 100).toFixed(1)}%</div>
      </div>
      
      <div className="recent-opponents">
        <h4>Recent Opponents</h4>
        <ul>
          {progress.recentOpponents.map((name, i) => (
            <li key={i}>{name}</li>
          ))}
        </ul>
      </div>
    </div>
  );
};
```

### Export/Import UI

```tsx
// src/components/NetworkManager.tsx

const NetworkManager: React.FC<{ mechConfig: MechConfig }> = ({ mechConfig }) => {
  const [network, setNetwork] = useState<tf.LayersModel | null>(null);
  const serializer = useRef(new NetworkSerializer());

  const handleExport = async () => {
    if (!network) return;
    
    const blob = await serializer.current.export(network, mechConfig, trainingMetrics);
    
    // Trigger download
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${mechConfig.name}.mecha-ai`;
    a.click();
    URL.revokeObjectURL(url);
  };

  const handleImport = async (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (!file) return;

    try {
      const model = await serializer.current.import(file, mechConfig);
      setNetwork(model);
    } catch (error) {
      // Handle incompatibility error
      alert('This AI file is not compatible with the current mech configuration.');
    }
  };

  return (
    <div className="network-manager">
      <input
        type="file"
        accept=".mecha-ai"
        onChange={handleImport}
      />
      <button onClick={handleExport} disabled={!network}>
        Export Trained AI
      </button>
    </div>
  );
};
```

---

## Web Worker Training (Performance)

For UI responsiveness, run training in a Web Worker.

```typescript
// src/workers/training.worker.ts

import * as tf from '@tensorflow/tfjs';
import { TrainingManager } from '../ai/TrainingManager';

self.onmessage = async (event) => {
  const { mechConfig, trainingConfig, opponents } = event.data;

  // Enable WebGL backend in worker if available
  await tf.setBackend('webgl');

  const manager = new TrainingManager(mechConfig, trainingConfig);
  
  const result = await manager.runTrainingSession(opponents, (progress) => {
    self.postMessage({ type: 'progress', data: progress });
  });

  // Serialize model for transfer back to main thread
  const serializedModel = await serializeModelForTransfer(result.model);
  
  self.postMessage({ 
    type: 'complete', 
    data: { model: serializedModel, metrics: result.metrics }
  });
};
```

```typescript
// src/hooks/useTrainingWorker.ts

const useTrainingWorker = () => {
  const workerRef = useRef<Worker | null>(null);

  const startTraining = useCallback((
    mechConfig: MechConfig,
    trainingConfig: TrainingConfig,
    opponents: MechConfig[],
    onProgress: (p: TrainingProgress) => void,
    onComplete: (model: tf.LayersModel, metrics: TrainingMetrics) => void
  ) => {
    workerRef.current = new Worker(
      new URL('../workers/training.worker.ts', import.meta.url)
    );

    workerRef.current.onmessage = async (event) => {
      if (event.data.type === 'progress') {
        onProgress(event.data.data);
      } else if (event.data.type === 'complete') {
        const model = await deserializeModel(event.data.data.model);
        onComplete(model, event.data.data.metrics);
      }
    };

    workerRef.current.postMessage({ mechConfig, trainingConfig, opponents });
  }, []);

  return { startTraining };
};
```

---

## Implementation Checklist

### Phase 1: Foundation
- [ ] Install TensorFlow.js dependency
- [ ] Create `src/ai/` directory structure
- [ ] Map existing mech stats to input tensor format
- [ ] Map existing action types to output tensor format
- [ ] Implement basic `NetworkBuilder` class

### Phase 2: Training System
- [ ] Implement `TrainingManager` class
- [ ] Integrate with existing simulation engine
- [ ] Implement reward function with game-specific tuning
- [ ] Add experience replay buffer
- [ ] Implement exploration/exploitation scheduling

### Phase 3: Serialization
- [ ] Implement `NetworkSerializer` export
- [ ] Implement `NetworkSerializer` import
- [ ] Add mech config hashing for compatibility checks
- [ ] Create `.mecha-ai` file format handler

### Phase 4: UI Integration
- [ ] Create `MechaTraining` component
- [ ] Create `TrainingProgressDisplay` component
- [ ] Create `NetworkManager` for export/import
- [ ] Add training UI to mech workshop/garage

### Phase 5: Tournament Integration
- [ ] Implement `TournamentBundler`
- [ ] Update tournament entry flow to include AI file
- [ ] Add AI validation on tournament server
- [ ] Enable replay rendering with neural network decisions

### Phase 6: Optimization
- [ ] Move training to Web Worker
- [ ] Implement batch inference for simulation speedup
- [ ] Add training session save/resume capability
- [ ] Profile and optimize tensor operations

---

## Codebase Integration Points

The following must be resolved by referencing the existing codebase:

| Integration Point | What to Find | Where to Look |
|-------------------|--------------|---------------|
| Mech parameters | All stats that affect combat | Mech config types/interfaces |
| Action types | Available actions per mech | Action enum or ability system |
| Simulation step | How battle tick/frame works | Existing simulation engine |
| Deterministic RNG | Seeded random for reproducibility | Existing sim or create new |
| Opponent pool | How to fetch training opponents | Matchmaking or mech database |
| Tournament entry | Existing entry submission flow | Tournament module |

---

## Notes

- The 5-point persona system can remain as a **fallback** or **baseline opponent** for training variety
- Network architecture parameters (layer sizes, activation functions) can be tuned based on training results
- Consider adding **curriculum learning**: start training against weaker opponents, gradually increase difficulty
- The `.mecha-ai` file format is versioned to support future architecture changes
