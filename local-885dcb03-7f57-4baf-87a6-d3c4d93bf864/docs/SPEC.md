# Samurai Squares

## Meta

**Game Name:** Samurai Squares
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Two players alternate moving a single Samurai on an 8x8 grid; moves are to adjacent empty cells via click/drag or tap; each round the AI places one impeding shield token to slow progress; victory is achieved by reaching the opponent's back rank or by capturing the opponent's Samurai.
**Session Minutes:**
  - 5
  - 15

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 8
    - 8
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
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 8x8 cells; each cell stores occupantId|null; occupantId: 1=Player Samurai, 2=AI Samurai, 3=Shield token
    **Rows:** 8
    **Cols:** 8
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
    **Last Move:**


## Mechanics

**Setup:**
  **Initial Placement:** Human Samurai placed at (7,3); AI Samurai placed at (0,4). No shields initially. Each round, the AI will place one Shield token on an empty cell to slow progress.
  **Starting Player Rule:** human
**Move Validation:**
  **Preconditions:**
    **Current Player:** must correspond to the active player (1 or 2)
    **Selected Row Col:** 0 <= row < 8, 0 <= col < 8
    **Target Cell:** must be adjacent to the current Samurai position and empty
  **Validity Checks:**
    - ownership_check
    - adjacency_check
    - occupancy_check
    - shield_block_check
  **Validation Algorithm:**
    **Name:** adjacency_check
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
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Identify current piece for current_player
      - Compute delta to target (row, col)
      - Verify target is one-step away in allowed directions
      - Ensure target cell is empty and not blocked by a shield
      - Return validity and preview data
**Movement:**
  **Allowed:** step
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture mechanic defined beyond reaching objectives; Human wins by reaching back rank or capturing AI via move rules.
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**

    **Parameters:**
      **Directions:**
        - (None)
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** End game after two consecutive passes
**Scoring:**
  **Method:** objectives
  **Winner Determination:** Human wins by occupying any cell on row 0 or by capturing AI Samurai; AI wins by occupying row 7 or by preventing human from achieving objectives; draw if two consecutive passes or no legal moves persist

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - from_row
    - from_col
    - to_row
    - to_col
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply move validation and, if valid, perform move; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using minimax; apply move validation and update state; place one shield token; switch to human; check end conditions
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
**Text:** AI with difficulty 'easy' MUST use a simple one-step-lookahead algorithm.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth minimax with pruning.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced minimax/mcts hybrid with deeper search.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_victory_by_back_rank_or_capture
  **Check Logic:** If Player Samurai occupies row 0 or AI Samurai is captured by the human
  **Priority:** immediate
**Lose:**
  **Condition:** ai_victory_by_back_rank_or_capture
  **Check Logic:** If AI Samurai occupies row 7 or Human Samurai is captured
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_or_two_consecutive_passes
  **Check Logic:** If both players have no legal moves or two passes occur in sequence
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to move
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
    **Player 1:** #1e90ff
    **Player 2:** #cccccc
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


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates deterministically and AI turn begins


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm and moves within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's imminent winning move when possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True