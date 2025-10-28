# King’s Courier

## Meta

**Game Name:** King’s Courier
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** On a 7x7 board, deliver courier tokens from start to throne by sliding or tapping to adjacent empties. The AI occasionally sends a rival guard to chase and block the most promising route.
**Session Minutes:**
  - 5
  - 15

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 7
    - 7
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
    **Grid:** 2D array of 7x7 cells; each cell may hold a piece id or null
    **Rows:** 7
    **Cols:** 7
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Last Move:**

  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Courier tokens placed at the human start area; throne located at the far end; AI rival guard starts near the AI side; all other squares empty.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination square must be empty
    - destination must be exactly one step away in one of the four cardinal directions
  **Validation Algorithm:**
    **Name:** grid_scan
    **Inputs:**
      - row
      - col
      - current_player
      - board
    **Outputs:**
      **Is Valid:** True
      **Preview:**

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
      - Confirm destination is within one-step move from a movable piece
      - Confirm destination cell is empty
      - Validate move against game rules and current state
      - Return is_valid and optional move preview
**Movement:**
  **Allowed:** step|slide
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
  **Result:** No captures occur in this game; guards block routes instead of capturing
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Affected Count:** 0
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
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** Human wins if courier reaches the throne; otherwise, determine winner by objective progress or declare draw if stalemate persists

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - from
    - to
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
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
**Text:** All game actions MUST be deterministic given the seed and event log; invalid actions MUST return a clear error with code and message.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic-based move selection.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a constrained search (minimax with pruning) to depth 4.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search (minimax with iterative deepening or MCTS) with seedable randomness for reproducibility.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_courier_reaches_throne
  **Check Logic:** Courier token on throne tile after human move
  **Priority:** immediate
**Lose:**
  **Condition:** ai_rival_guard_catches_courier or human_no_moves
  **Check Logic:** Guard occupies courier tile or no legal human moves
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_or stalemate after a full round
  **Check Logic:** No moves available for both sides
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


**Given:** A valid capture exists per game rules
**When:** Player performs a capturing move
**Then:** Captured pieces/territory updated exactly as defined


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute winning move


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's winning move


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True