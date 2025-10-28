# Gridlock Grand Prix

## Meta

**Game Name:** Gridlock Grand Prix
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Click a token to move to an adjacent empty cell or drag along a straight path toward a finish line; AI drivers place roadblocks to slow opponents.
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
    **Grid:** 8x8 grid with coordinates; each cell may hold a piece or be empty
    **Rows:** 8
    **Cols:** 8
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
    **Last Move:** None

## Mechanics

**Setup:**
  **Initial Placement:** Human tokens placed on their starting edge; AI tokens placed on the opposite edge. Each player starts with a fixed number of tokens (e.g., 4) distributed along their starting row.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - ownership_check: action must target a token owned by current_player
    - adjacency_or_path_check: destination must be adjacent or reachable via a straight path without obstacles
    - destination_empty: target cell must be empty
    - block_visibility_check: no hidden blocks on path
  **Validation Algorithm:**
    **Name:** path_and_adjacency_check
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
      **Min Distance:** 1
      **Range:** any
      **Require Bounded:** False
    **Steps:**
      - Confirm current_player owns the token at source
      - If destination is adjacent, allow when empty
      - If destination is further along a straight line, ensure all intermediate cells are empty and path is straight
      - Return is_valid and preview of move
**Movement:**
  **Allowed:** step|slide
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** any
**Capture:**
  **Type:** block_placement
  **Directions:**
    - up
    - down
    - left
    - right
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** AI may place roadblocks on empty cells to slow or block opposing tokens
  **Capture Algorithm:**
    **Name:** apply_roadblocks
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Blocked Count:** int
    **Parameters:**
      **Directions:**
        - up
        - down
        - left
        - right
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - Mark selected cells as blocked for the remainder of the game or until cleared by a rule
      - Update affected movement paths for affected tokens
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** The first player to reach their finish line wins; if both reach on the same turn, compare points; if neither reaches finish, higher point total wins; draw if tied and no further moves

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - from
    - to
  **Condition:** current_player == 1 AND game.status == playing AND moves_are_legal
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
**Text:** Clear, testable rule with input/output MUST define a deterministic input/output behavior.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic that places blocks and moves tokens with minimal lookahead.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate heuristic with limited search depth.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced planning strategy with deeper search and pruning.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_finish_reached
  **Check Logic:** Any human token occupies any human finish line cell
  **Priority:** immediate
**Lose:**
  **Condition:** ai_finish_reached
  **Check Logic:** Any AI token occupies its finish line cell
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_or_time_out
  **Check Logic:** At start of a turn, if current player has no legal moves and the opponent also has none, declare draw
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
    **Player 1:** #1e90ff
    **Player 2:** #ff4500
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