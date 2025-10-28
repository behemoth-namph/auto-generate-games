# Maze Runner Grid

## Meta

**Game Name:** Maze Runner Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A deterministic turn-based maze puzzle on a 9x9 grid where the human runner seeks a path to the exit by rotating and placing path tiles. The AI intermittently adds wandering walls to complicate connectivity, requiring the human to adapt the route in real-time.
**Session Minutes:**
  - 5
  - 15

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 9
    - 9
  **Neighbors:** for grid: ['up','down','left','right']
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #000000
    **Pieces Played:** 0
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 9x9 array of path/wall tiles; each cell stores tile type, rotation, and occupancy
    **Rows:** 9
    **Cols:** 9
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Last Move:** None
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Runner token placed at (0,0); exit at (8,8); initial path tiles arranged to form at least one valid route; AI wandering walls inactive until after the first few human moves
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - placement maintains at least one valid path from runner to exit after the move
    - tile rotation is within allowed states
    - no overlapping tiles or tiles placed outside the 9x9 grid
  **Validation Algorithm:**
    **Name:** path_check
    **Inputs:**
      - row
      - col
      - current_player
      - board
    **Outputs:**
      **Is Valid:** boolean
      **Preview:** object
    **Parameters:**
      **Directions:**
        - up
        - down
        - left
        - right
      **Min Chain Length:** 1
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Identify target cell; ensure empty slot
      - Simulate tile placement/rotation
      - Compute connectivity from runner to exit
      - If connected, return is_valid = true with a preview of resulting path
**Movement:**
  **Allowed:** placement|rotate
  **Directions:**
    - rotate_tile
    - drag_to_place
  **Range:** 1
**Capture:**
  **Type:** area_conversion
  **Directions:**
    - up
    - down
    - left
    - right
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 2
  **Result:** AI wandering walls are added to random empty tiles; paths may be closed or opened
  **Capture Algorithm:**
    **Name:** apply_wandering_walls
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Affected Count:** int
    **Parameters:**
      **Directions:**
        - up
        - down
        - left
        - right
      **Min Chain Length:** 2
      **Require Bounded:** False
    **Steps:**
      - Increment move counter to determine wandering interval
      - Choose candidate empty cells based on a simple heuristic
      - Place wall tiles in chosen cells
      - Update board state and recalculate connectivity where needed
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** path_connectivity
  **Winner Determination:** If human runner reaches exit, human wins; if exit becomes permanently unreachable due to walls and AI cannot create a new valid path, AI wins; draw if no legal moves exist for both players during a full round

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - tile_id
    - rotation
    - target_row
    - target_col
  **Condition:** current_player == 1 AND game.status == playing AND [move validity conditions]
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic without deep search.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth search strategy.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning and lookahead.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_reaches_exit
  **Check Logic:** If the runner token occupies the exit cell (8,8) on human turn or after a valid move
  **Priority:** immediate
**Lose:**
  **Condition:** ai_blocks_all_paths
  **Check Logic:** If there exists no valid path from runner to exit and human has exhausted legal moves
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_after_round
  **Check Logic:** If after a full cycle both players have no legal moves
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to place or rotate a tile
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser/draw message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #000000
    **Player 2:** #ffffff
    **Outline:** #aaaaaa
  **Valid Move Highlight:**
    **Enabled:** True
    **Color:** #66ccff
  **Last Move Highlight:**
    **Enabled:** True
    **Color:** #ffcc00
  **Flip Animation:**
    **Enabled:** True
    **Duration Ms:** 180

## Technical

**Platform:** HTML5 Canvas
**Performance:**
  **Target Fps:** 30
  **Max Load Time Ms:** 1000
  **Max Ai Think Time Ms:** 2000

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid placement exists per game rules
**When:** Player places or rotates a tile
**Then:** Board updates accordingly and path validity is re-evaluated


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic and returns a valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning or blocking move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt to block human's winning path


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True