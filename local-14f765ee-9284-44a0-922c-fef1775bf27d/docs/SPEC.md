# Farm Field Flip

## Meta

**Game Name:** Farm Field Flip
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** On a 6x6 grid, players flip or slide crop tokens to align full rows of identical vegetables while the AI farmer introduces pests as obstacles to avoid.
**Session Minutes:**
  - 10
  - 20

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 6
    - 6
  **Neighbors:**
    - up
    - down
    - left
    - right
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #228B22
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
    **Grid:** 2D array of cells with occupantId or null and tile type (crop or pest)
    **Rows:** 6
    **Cols:** 6
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

## Mechanics

**Setup:**
  **Initial Placement:** Starting distribution of crop tokens across the 6x6 grid is provided by deterministic seed; no pests present at start.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination is within board bounds
    - destination is adjacent for slide or is the same cell for a flip action target
    - target cell is empty for slides; flip targets a tile to a different vegetable type
    - flip result must be a valid vegetable state (e.g., cycle within allowed types)
  **Validation Algorithm:**
    **Name:** adjacency_check
    **Inputs:**
      - row
      - col
      - current_player
      - board
    **Outputs:**
      **Is Valid:** True
      **Preview:** object or array if applicable
    **Parameters:**
      **Directions:**
        - up
        - down
        - left
        - right
      **Min Chain Length:** 0
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Clamp coordinates to board bounds
      - Verify destination is adjacent for slide or valid flip target
      - Ensure destination cell is empty (for slide) or that flip result is allowed
      - If flip, compute next vegetable type and ensure legality
**Movement:**
  **Allowed:** slide|flip
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1 for both slide and flip (single-tile interactions)
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture events; progress driven by pattern completion and pest placement
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**

    **Parameters:**

    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** pattern_completion
  **Winner Determination:** Human wins if any row (6 cells) contains identical crop tokens with no pests in that row; AI wins if a full row of Pest obstacles appears or if draw conditions occur without human victory

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - actionType (slide|flip)
    - targetRow|targetCol (for slide)
  **Condition:** current_player == 1 AND game.status == playing AND a legal move exists
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
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple deterministic pest-placement heuristic.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate heuristic balancing pest placement and block strategies.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced planning algorithm with lookahead.
**Type:** core


## End Conditions

**Win:**
  **Condition:** Human completes a full row of identical vegetables with no pests in that row
  **Check Logic:** At end of a turn, scan all 6 rows; if any row contains six tokens of the same vegetable type and zero Pest tokens, human_wins
  **Priority:** immediate
**Lose:**
  **Condition:** AI forms a full Pest row (one entire row occupied by Pest tokens)
  **Check Logic:** Scan all rows; if any row consists entirely of Pest tokens, ai_wins
  **Priority:** immediate
  **Condition:** Time or no legal moves lead to end
  **Check Logic:** If both players have no legal moves for a full round, declare draw unless a win condition is met
  **Priority:** end_turn
**Draw:**
  **Condition:** No legal moves exist for both players for an entire round
  **Check Logic:** After a full pass cycle with no changes, trigger draw
  **Priority:** end_turn
  **Condition:** Move limit reached without a winner
  **Check Logic:** If move_count >= 200, declare draw
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to perform a move
  - Drag to slide, tap to flip, hover for previews
  - Keyboard navigation with ARIA focus
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Crop Type 1:** #4CAF50
    **Crop Type 2:** #8BC34A
    **Crop Type 3:** #FFC107
    **Crop Type 4:** #9C27B0
    **Pest:** #555555
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
**Accessibility:**
  **Aria Labels:** True
  **Aria Live:** True
**Modularity:**
  **Ui Split:** True

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates deterministically; last_move recorded; AI turn begins


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple deterministic pest-placement within 2 seconds


**Given:** Human can win by row completion
**When:** Human achieves a full matching row
**Then:** Human wins and modal displays


**Given:** AI blocks all winning paths
**When:** AI forms a full Pest row
**Then:** AI wins and modal displays


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True