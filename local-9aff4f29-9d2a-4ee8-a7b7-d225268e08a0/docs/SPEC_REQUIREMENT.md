# 🎯 Puzzle Game Requirements Specification (v1.0)

**Game Type:** Puzzle Games (2048, Sudoku, Sliding Puzzles, Sokoban, etc.)

**Purpose:** Produce complete, deterministic, testable requirements for single-player puzzle games with goal-oriented mechanics, move optimization, and progressive difficulty.

---

## 🔴 0. Meta Rules (CORE)

- Use normative language (MUST / SHOULD / MAY)
- All game logic MUST be deterministic and replayable
- UI and gameplay MUST be decoupled
- Puzzles MUST have solvable states
- If a feature does not exist, explicitly state: "This game does not use "

---

## 🔴 1. Scope & Core Feature Declaration (CORE)

### **Puzzle Games Typically Use:**

| Feature | Value | Description |
|---------|-------|-------------|
| **Board/Grid System** | **REQUIRED** | Grid or spatial area for puzzle state |
| **Piece/Tile System** | **REQUIRED** | Movable/interactable elements |
| **Holdings** | **NOT USED** | No hands/decks (exceptions: inventory puzzles) |
| **Allegiance** | **NOT USED** | Single player, no opponents |
| **Turn Mode** | **PLAYER-DRIVEN** | No enforced turns, player controls pacing |
| **Randomness** | **CONTROLLED** | Initial state may be random, gameplay deterministic |
| **Physics Context** | **GRID-BASED** or **GRAVITY** | Depends on puzzle type |
| **Goal System** | **REQUIRED** | Clear win condition (target state) |
| **Move Counting** | **REQUIRED** | Track moves for scoring/optimization |
| **Undo System** | **REQUIRED** | Allow move reversal |

### ✅ Puzzle Game Specific Assertions:

1. Puzzle MUST have defined goal state
2. Initial state MUST be solvable (or explicitly unsolvable for challenge)
3. Move validation MUST be deterministic
4. Undo MUST restore exact previous state
5. Score/rating based on moves, time, or both

---

## 🔴 2. Core Mechanics & Action Legality (CORE)

### **Puzzle Game Actions:**

| Action Type | Examples | Validation |
|-------------|----------|------------|
| **Slide** | 2048, Sliding Puzzle | Direction legal, tiles can move |
| **Swap** | Bejeweled, Candy Crush | Adjacent tiles, creates match |
| **Place** | Sudoku | Cell empty, number not in row/col/box |
| **Push** | Sokoban | Player can push, no wall behind |
| **Rotate** | Tetris | Piece fits after rotation |
| **Fill** | Flood Fill, Minesweeper | Cell unvisited, rule satisfied |

### **Action Structure:**

```javascript
{
  "action": "slide",
  "direction": "up" | "down" | "left" | "right",
  "preconditions": [
    "grid is not in goal state",
    "tiles exist that can move in direction"
  ],
  "postconditions": [
    "all movable tiles shifted",
    "new tile spawned (if applicable)",
    "move count incremented",
    "check for goal state"
  ],
  "illegal_cases": [
    "no tiles can move in direction",
    "game already won/lost"
  ]
}
```

### **Common Patterns:**

1. **Merge on Collision (2048):**
   - Slide tiles in direction
   - Tiles with same value merge
   - New tile spawns in empty cell

2. **Constraint Satisfaction (Sudoku):**
   - Place number in cell
   - Check row/column/box constraints
   - Validate no duplicates

3. **Path Finding (Sokoban):**
   - Move player character
   - Push boxes to targets
   - Cannot pull, only push

### ✅ Deadlock Prevention:

- Check if puzzle still solvable after each move
- Detect "no legal moves" state
- Provide hint system or undo to escape deadlock

---

## 🔴 3. Board & Spatial Context (CORE)

### **Puzzle Board Definition:**

```json
{
  "grid": {
    "type": "2d_grid" | "hexagonal" | "irregular",
    "rows": 4,
    "cols": 4,
    "cells": [
      {
        "id": "cell_0_0",
        "row": 0,
        "col": 0,
        "value": 2 | null,
        "state": "empty" | "filled" | "locked",
        "is_target": false,
        "is_movable": true
      }
    ],
    "physics": {
      "gravity": false | "down" | "custom",
      "merge_on_collision": true,
      "wrap_edges": false
    }
  }
}
```

### **Spatial Rules by Puzzle Type:**

#### **2048-Style:**
- 4×4 grid (typically)
- Tiles slide until collision
- Merge on same-value collision
- Gravity: none (or directional on slide)

#### **Sudoku:**
- 9×9 grid divided into 3×3 boxes
- Each cell: 1-9 or empty
- No physics, pure constraint checking

#### **Sokoban:**
- Variable grid size
- Player position + box positions
- Walls are immovable
- Targets are goal positions

#### **Sliding Puzzle:**
- N×N grid with one empty space
- Tiles slide into empty space
- Goal: specific arrangement

### **Movement Physics:**

```javascript
function slideDirection(grid, direction) {
  // Apply physics based on puzzle type
  switch (puzzleType) {
    case "2048":
      return slide_and_merge(grid, direction);
    case "gravity_puzzle":
      return apply_gravity(grid, direction);
    case "sliding":
      return slide_into_empty(grid, direction);
  }
}
```

---

## 🔴 4. Goal & Win Conditions (CORE)

### **Puzzle Goal Definitions:**

| Puzzle Type | Goal | Validation |
|-------------|------|------------|
| **2048** | Reach 2048 tile | Any cell.value >= 2048 |
| **Sudoku** | Fill all cells legally | No empty cells, all constraints satisfied |
| **Sliding Puzzle** | Arrange in order | grid matches target_state |
| **Sokoban** | All boxes on targets | every box.position in target_positions |
| **Flood Fill** | Fill all cells | all cells.state === "filled" |

### **Goal State Specification:**

```json
{
  "goal": {
    "type": "reach_value" | "match_pattern" | "fill_all" | "arrange_order",
    "condition": {
      "target_value": 2048,
      "target_pattern": [[1,2,3],[4,5,6],[7,8,null]],
      "all_cells_filled": true,
      "boxes_on_targets": true
    },
    "check_frequency": "after_each_move",
    "win_message": "Congratulations! You reached 2048!"
  }
}
```

### **Loss Conditions:**

```json
{
  "lose": {
    "type": "no_legal_moves" | "time_expired" | "lives_depleted",
    "conditions": [
      {
        "check": "no_valid_moves_remaining",
        "message": "No more moves available!"
      },
      {
        "check": "timer <= 0",
        "message": "Time's up!"
      }
    ]
  }
}
```

### **Optimal Solution Tracking:**

```javascript
{
  "scoring": {
    "optimal_moves": 24,
    "player_moves": 32,
    "time_taken_sec": 145,
    "rating": "3_stars", // based on moves/time
    "hints_used": 2
  }
}
```

---

## 🔴 5. Turn/Move Sequencer (CORE)

### **Puzzle Game Flow:**

Unlike turn-based games, puzzles are **continuous player-driven**:

```
Game Start
    ↓
Player Input (any time)
    ↓
Validate Move
    ↓
Apply Move
    ↓
Update State (merge, gravity, etc.)
    ↓
Check Goal
    ↓
  (goal reached) → Win Screen
  (no moves left) → Lose Screen
  (continue) → Wait for Player Input
```

### **No Time Pressure (Classic Mode):**
- Player can take unlimited time
- No countdown timer
- Focus on move optimization

### **Timed Mode (Optional):**
- Countdown timer
- Lose if timer expires
- Time bonus for quick completion

### **Move History:**

```javascript
{
  "moves": [
    {
      "move_number": 1,
      "action": "slide",
      "direction": "up",
      "state_before": { /* grid snapshot */ },
      "state_after": { /* grid snapshot */ },
      "timestamp": 1699564800000
    }
  ]
}
```

---

## 🔴 6. Undo/Redo System (CORE for Puzzles)

### **Undo Requirements:**

Puzzles MUST support undo:

```javascript
{
  "undo": {
    "enabled": true,
    "max_undo_depth": "unlimited" | 10,
    "cost": "none" | "score_penalty",
    "restores": [
      "grid_state",
      "move_count",
      "score",
      "timer (if applicable)"
    ]
  }
}
```

### **Implementation:**

```javascript
class PuzzleGame {
  constructor() {
    this.stateHistory = [];
    this.currentStateIndex = -1;
  }
  
  makeMove(action) {
    // Save current state
    this.stateHistory.push(cloneDeep(this.state));
    this.currentStateIndex++;
    
    // Apply move
    this.state = this.applyAction(this.state, action);
    
    // Truncate redo history
    this.stateHistory = this.stateHistory.slice(0, this.currentStateIndex + 1);
  }
  
  undo() {
    if (this.currentStateIndex > 0) {
      this.currentStateIndex--;
      this.state = this.stateHistory[this.currentStateIndex];
    }
  }
  
  redo() {
    if (this.currentStateIndex < this.stateHistory.length - 1) {
      this.currentStateIndex++;
      this.state = this.stateHistory[this.currentStateIndex];
    }
  }
}
```

---

## 🟡 7. Randomness & Replay Integrity (IMPORTANT)

### **Puzzle Randomness:**

**Initial State Generation:**

```javascript
{
  "rng": {
    "algorithm": "xoshiro256**",
    "seed": "puzzle-12345",
    "uses": [
      {
        "phase": "initialization",
        "purpose": "generate_starting_grid",
        "ensures": "puzzle_is_solvable"
      },
      {
        "phase": "new_tile_spawn", // 2048
        "purpose": "select_empty_cell_and_value",
        "distribution": { "2": 0.9, "4": 0.1 }
      }
    ]
  }
}
```

### **Deterministic Gameplay:**

After initialization, gameplay MUST be deterministic:

- Player actions fully determine outcome
- No random events during solving
- Exception: some puzzles spawn new elements (2048)
  - This MUST be seeded and reproducible

### **Replay:**

```javascript
{
  "replay": {
    "seed": "puzzle-12345",
    "initial_state": { /* generated grid */ },
    "moves": [
      { "move": 1, "action": "slide", "direction": "up" },
      { "move": 2, "action": "slide", "direction": "left" }
    ],
    "final_state": { /* ending grid */ },
    "outcome": "win" | "lose"
  }
}
```

Replaying moves with same seed MUST produce identical outcome.

---

## 🟡 8. Hint System (IMPORTANT for Puzzles)

### **Hint Mechanics:**

```json
{
  "hints": {
    "available": true,
    "max_hints": 3 | "unlimited",
    "cost": "score_penalty" | "time_penalty" | "none",
    "types": [
      {
        "type": "next_move",
        "shows": "optimal_next_action",
        "visual": "highlight cell or direction"
      },
      {
        "type": "mistake",
        "shows": "incorrect_placement",
        "visual": "mark error in red"
      },
      {
        "type": "solution_path",
        "shows": "sequence_of_moves",
        "visual": "show first 3 moves"
      }
    ]
  }
}
```

### **Hint Implementation:**

```javascript
function getHint(current_state) {
  // Run solver to find optimal path
  const solution = solve(current_state);
  
  if (solution.exists) {
    return {
      suggested_action: solution.next_move,
      explanation: "This move leads toward solution",
      confidence: 0.95
    };
  } else {
    return {
      suggested_action: "undo",
      explanation: "Puzzle may be in unsolvable state",
      confidence: 1.0
    };
  }
}
```

---

## 🟢 9. Difficulty & Progression (OPTIONAL)

### **Difficulty Levels:**

```json
{
  "difficulty": {
    "levels": [
      {
        "name": "Easy",
        "grid_size": "4x4",
        "starting_tiles": 4,
        "time_limit": null,
        "hints_available": "unlimited"
      },
      {
        "name": "Medium",
        "grid_size": "5x5",
        "starting_tiles": 6,
        "time_limit": 300,
        "hints_available": 3
      },
      {
        "name": "Hard",
        "grid_size": "6x6",
        "starting_tiles": 8,
        "time_limit": 180,
        "hints_available": 1
      }
    ]
  }
}
```

### **Progressive Unlocking:**

```javascript
{
  "progression": {
    "type": "level_based",
    "levels": [
      {
        "level_id": 1,
        "unlocked_by": "default",
        "puzzle_config": { /* level 1 settings */ }
      },
      {
        "level_id": 2,
        "unlocked_by": "complete_level_1",
        "puzzle_config": { /* level 2 settings */ }
      }
    ]
  }
}
```

---

## 🟢 10. Interface Visibility & Indicators (REQUIRED)

### **Puzzle Game UI Requirements:**

```
┌─────────────────────────────────┐
│  SCORE: 1024    MOVES: 32       │
│  BEST: 2048     TIME: 02:45     │
│  [Undo] [Redo] [Hint] [Restart] │
├─────────────────────────────────┤
│                                 │
│          PUZZLE GRID            │
│      [ Game State Display ]     │
│                                 │
├─────────────────────────────────┤
│  Next: 🎲  Hints Left: 2/3      │
└─────────────────────────────────┘
```

#### **Essential Indicators:**

- [ ] Current score/progress
- [ ] Move counter
- [ ] Best score (if applicable)
- [ ] Timer (if timed mode)
- [ ] Hints remaining
- [ ] Undo/Redo availability

#### **Feedback Elements:**

- [ ] Valid move indicator (glow/highlight)
- [ ] Invalid move feedback (shake/red flash)
- [ ] Merge/combine animation
- [ ] Victory animation
- [ ] Loss screen

---

## 🟢 11. Scoring Systems (OPTIONAL)

### **Common Scoring Methods:**

| Method | Description | Example |
|--------|-------------|---------|
| **Move-Based** | Fewer moves = higher score | Sokoban, Sliding Puzzle |
| **Value-Based** | Tile values accumulate | 2048 |
| **Time-Based** | Faster completion = bonus | Timed Sudoku |
| **Star Rating** | 1-3 stars based on performance | Mobile puzzle games |
| **Combo System** | Consecutive actions = multiplier | Match-3 puzzles |

### **Score Calculation:**

```javascript
function calculateScore(puzzle_state) {
  let score = 0;
  
  // Base score from tile values (2048)
  score += sum(grid.tiles.map(t => t.value));
  
  // Move efficiency bonus
  if (moves < optimal_moves * 1.2) {
    score += 500; // efficiency bonus
  }
  
  // Time bonus (if timed)
  if (time_remaining > 0) {
    score += time_remaining * 10;
  }
  
  // Hint penalty
  score -= hints_used * 100;
  
  return Math.max(0, score);
}
```

---

## 🔴 12. Puzzle Validation & Solvability (CORE)

### **Solvability Requirements:**

Every generated puzzle MUST be validated:

```javascript
function isPuzzleSolvable(initial_state) {
  // Use appropriate solver algorithm
  switch (puzzle_type) {
    case "sliding_puzzle":
      return hasValidInversionParity(initial_state);
    
    case "sudoku":
      return backtrackSolver(initial_state) !== null;
    
    case "sokoban":
      return BFS_search(initial_state, goal_state) !== null;
    
    case "2048":
      // Always solvable unless board full with no merges
      return hasLegalMoves(initial_state);
  }
}
```

### **Generation with Validation:**

```javascript
function generatePuzzle(difficulty) {
  let puzzle;
  let attempts = 0;
  
  do {
    puzzle = randomGenerate(difficulty);
    attempts++;
  } while (!isPuzzleSolvable(puzzle) && attempts < 100);
  
  if (!isPuzzleSolvable(puzzle)) {
    throw new Error("Failed to generate solvable puzzle");
  }
  
  return puzzle;
}
```

---

## 🔴 13. Logging & Replay Requirements (CORE)

### **Puzzle Game Logs:**

```json
{
  "puzzle_id": "2048-easy-12345",
  "puzzle_type": "2048",
  "seed": "12345-timestamp",
  "difficulty": "easy",
  "initial_state": {
    "grid": [[2,null,null,null],[null,2,null,null],[null,null,null,null],[null,null,null,null]]
  },
  "moves": [
    {
      "move_number": 1,
      "action": "slide",
      "direction": "up",
      "tiles_moved": 2,
      "merges": [],
      "new_tile": { "position": [3,1], "value": 2 },
      "score_delta": 0
    },
    {
      "move_number": 2,
      "action": "slide",
      "direction": "left",
      "tiles_moved": 3,
      "merges": [{ "position": [0,0], "from_values": [2,2], "to_value": 4 }],
      "new_tile": { "position": [2,3], "value": 2 },
      "score_delta": 4
    }
  ],
  "final_state": {
    "grid": [/* final grid */],
    "status": "win",
    "score": 2048,
    "moves": 127,
    "time": 342,
    "hints_used": 1
  }
}
```

### **Replay Validation:**

```javascript
function replayPuzzle(log) {
  let state = initPuzzle(log.seed);
  assert(state === log.initial_state);
  
  for (let move of log.moves) {
    state = applyMove(state, move);
  }
  
  assert(state === log.final_state);
  assert(state.status === log.final_state.status);
}
```

---

## 🧩 UI Architecture Rules (Puzzle Games)

### **1. Component Structure:**

```
/components/
  /PuzzleGame/
    PuzzleGrid.tsx       - Main grid display
    Tile.tsx             - Individual tile/cell
    ScorePanel.tsx       - Score, moves, timer
    ControlPanel.tsx     - Undo, Redo, Hint, Restart
    HintOverlay.tsx      - Visual hint display
    VictoryModal.tsx     - Win screen
    GameOverModal.tsx    - Loss screen
```

### **2. State Management:**

```javascript
const puzzleState = {
  grid: [[2, 4, 2, 4], ...],
  score: 1024,
  moves: 32,
  bestScore: 2048,
  hintsRemaining: 2,
  status: "playing" | "won" | "lost",
  history: [/* previous states */]
};
```

### **3. Input Handling:**

```javascript
// Keyboard controls
document.addEventListener("keydown", (e) => {
  switch (e.key) {
    case "ArrowUp": makeMove("up"); break;
    case "ArrowDown": makeMove("down"); break;
    case "ArrowLeft": makeMove("left"); break;
    case "ArrowRight": makeMove("right"); break;
    case "z": if (e.ctrlKey) undo(); break;
    case "h": showHint(); break;
  }
});

// Touch/swipe controls
let touchStart = null;
gridElement.addEventListener("touchstart", (e) => {
  touchStart = { x: e.touches[0].clientX, y: e.touches[0].clientY };
});

gridElement.addEventListener("touchend", (e) => {
  const touchEnd = { x: e.changedTouches[0].clientX, y: e.changedTouches[0].clientY };
  const direction = getSwipeDirection(touchStart, touchEnd);
  if (direction) makeMove(direction);
});
```

### **4. Animations:**

Puzzle games heavily rely on animations:

- Tile slide/merge animations (200-300ms)
- New tile spawn (fade in + scale)
- Invalid move shake
- Victory confetti/celebration
- Smooth transitions between states

---

## 📊 Puzzle Game Checklist

### **Core Requirements:**

- [ ] Grid/board defined with clear dimensions
- [ ] Tile/piece movement rules explicit
- [ ] Goal state clearly defined
- [ ] Win condition deterministic
- [ ] Loss condition deterministic
- [ ] Move validation implemented
- [ ] Undo/redo system functional
- [ ] Hint system (optional but recommended)
- [ ] Puzzle is solvable

### **Implementation:**

- [ ] Grid state representation
- [ ] Move application function
- [ ] Goal state checker
- [ ] Legal moves generator
- [ ] Undo history management
- [ ] Hint algorithm (if applicable)
- [ ] Score calculator
- [ ] Replay system

### **UI:**

- [ ] Grid rendering
- [ ] Tile animations
- [ ] Score display
- [ ] Move counter
- [ ] Timer (if timed)
- [ ] Control buttons
- [ ] Hint visualization
- [ ] Victory/loss modals

---

## 🎯 Summary

Puzzle games are characterized by:

✅ **Single-Player** - No opponents  
✅ **Goal-Oriented** - Clear objective  
✅ **Deterministic** - Same moves = same outcome  
✅ **Undoable** - Allow experimentation  
✅ **Self-Paced** - Player controls timing  
✅ **Skill-Based** - Optimal solutions exist  

This specification ensures puzzle games are:
- **Fair** - Always solvable (or explicitly not)
- **Replayable** - Consistent behavior
- **Learnable** - Clear rules and feedback
- **Enjoyable** - Smooth UX with undo/hints
