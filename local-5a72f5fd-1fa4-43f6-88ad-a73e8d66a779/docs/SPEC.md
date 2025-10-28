# Pirate Pathboard

## Meta

**Game Name:** Pirate Pathboard
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Tap a pirate to select, then tap an adjacent empty cell to move or drag along a march; AI intermittently drops sea-storm tokens to impede progress towards treasure docks.
**Session Minutes:**
  - 10
  - 60

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
    **Id:** 2
    **Type:** ai
    **Color:** #ff5555
    **Pieces Played:** int
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array of cells; each cell holds occupantId|null and cell type
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

## Mechanics

**Setup:**
  **Initial Placement:** Human pirates occupy the bottom two rows; AI pirates occupy the top two rows. Treasure docks are placed on the opposing back edge. Each side starts with an equal number of pirates. Sea-storm tokens appear on the board as obstacles that may block paths or slow movement.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - player_owns_piece
    - destination_is_adjacent_or_contiguous_for_march
    - destination_is_within_bounds
    - destination_is_empty_or_contains_sea_storm_if_capture_allowed
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
      - Verify the selected piece belongs to current_player
      - Check destination is within bounds
      - If a march is dragged, verify all intermediate cells are empty
      - If destination is a dock, verify piece type can enter dock
      - If destination contains a sea-storm token, reject unless token is removed by a prior effect
**Movement:**
  **Allowed:** step|march
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** variable (1 step for tap-move; drag may specify multiple consecutive steps up to encountered obstacle or sea-storm limit)
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No captures occur by movement; sea-storm tokens are blocking obstacles, not captured pieces.
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
  **Extra Turn Conditions:** end game if two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** Winner is the first side to have a pirate occupy any treasure dock; if both occupy docks on the same turn, the player who achieved it first wins; otherwise, points are tallied by remaining distance to docks when no moves remain.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceId
    - destinationRow
    - destinationCol
  **Condition:** current_player == 1 AND game.status == playing
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; potentially trigger AI turn; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update state; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST be defined; violations MUST return {{ errorCode, message }}.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple random-valid-move strategy.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-minimax strategy with depth 4.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced predictive-minimax/MCTS hybrid with depth up to 6.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_reaches_dock
  **Check Logic:** Any human pirate occupies a treasure dock cell
  **Priority:** immediate
**Lose:**
  **Condition:** ai_reaches_dock
  **Check Logic:** Any AI pirate occupies a treasure dock cell
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_current_or_opponent
  **Check Logic:** If current player and opponent have zero legal moves on their turns
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Tap to select a pirate
  - Tap adjacent empty cell to move
  - Drag for march
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show Sea-storm token spawns
  - Display winner/defeat message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #1e90ff
    **Player 2:** #ff5555
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
**Aria:**
  **Labels Enabled:** True
  **Live Region:** polite

## Technical

**Platform:** HTML5 Canvas
**Performance:**
  **Target Fps:** 30
  **Max Load Time Ms:** 1000
  **Max Ai Think Time Ms:** 2000
**Input Methods:**
  - touch
  - mouse
**Assets:**
  **Sprites:** pirate tokens, docks, sea-storm tokens
  **Sounds:** movement, dock_reached, sea_storm_spawn

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid move exists
**When:** Player performs a moving action
**Then:** State updates deterministically and UI reflects move


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm and moves within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute winning move (or block human if applicable)


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's winning move when possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True