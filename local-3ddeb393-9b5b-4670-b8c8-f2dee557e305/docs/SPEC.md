# Coin Stack Circuit

## Meta

**Game Name:** Coin Stack Circuit
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A 6x6 grid where you sort coins into correct stacks by moving coins to adjacent empty cells or sliding within a row to form complete stacks. The AI occasionally spills extra coins that must be re-sorted.
**Session Minutes:**
  - 5
  - 60

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 6
    - 6
  **Neighbors:** directions: [up, down, left, right]
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #1e90ff
    **Pieces Played:** int
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Color:** #ff4500
    **Pieces Played:** int
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array of cells; each cell stores occupantId|null and stack information
    **Rows:** int
    **Cols:** int
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** int (1 or 2)
    **Status:** string (playing|human_wins|ai_wins|draw)
    **Move Count:** int
    **Last Move:** object
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Last Move:** None

## Mechanics

**Setup:**
  **Initial Placement:** Coins distributed across the 6x6 grid into color-coded stacks; stacks expect specific colors. AI may spill extra coins on its turns.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell must be empty
    - move must be to an adjacent cell or a legal row slide
    - move must not violate stack capacity or color constraints
  **Validation Algorithm:**
    **Name:** grid_scan
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
      - Identify target coordinates
      - Verify occupancy is empty
      - Check adjacency or row-slide legality
      - Check color/stack compatibility
      - Return validity and preview of resulting state
**Movement:**
  **Allowed:** slide|move
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1 (step) for individual moves; slides may traverse multiple cells within a row under rule checks
**Capture:**
  **Type:** area_conversion
  **Directions:**
    - row
    - column
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 1
  **Result:** AI spill events or re-sorting adjustments cause affected coins to re-align into their designated stacks
  **Capture Algorithm:**
    **Name:** apply_spill_and_rebalance
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
        - row
        - column
      **Min Chain Length:** 1
      **Require Bounded:** False
    **Steps:**
      - Detect spill trigger from AI action or valid move
      - Distribute spilled coins into available adjacent spots
      - Rebalance stacks to reflect new layout
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** count_pieces
  **Winner Determination:** Highest total of correctly stacked coins at end of game wins; in case of tie, draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - move_type
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
**Text:** Clear, testable rule with input/output (MUST / SHOULD / MAY).
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic: pick the first valid immediate improvement.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate minimax with depth 4 search.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning and lookahead and seedable randomness for replay.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_objectives_completed
  **Check Logic:** All coins are in their designated correct stacks; human has achieved the objective before AI
  **Priority:** immediate
**Lose:**
  **Condition:** ai_objectives_completed
  **Check Logic:** All coins are in their designated correct stacks; AI has achieved the objective before human
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_players
  **Check Logic:** During a full rotation, neither player has a legal move
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to make move
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #ffd700
    **Player 2:** #c0c0c0
    **Outline:** #888888
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


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates deterministically and UI reflects new board


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt to block human's winning path


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True