# Space Station Switch

## Meta

**Game Name:** Space Station Switch
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Airlock tokens must be routed to docking stations on an 8x8 grid. Players move a token to a neighboring empty cell by clicking, or drag along a straight line to an empty cell. The AI deploys a meteor token that temporarily blocks a cell to disrupt routes.
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
  **Neighbors:** ['up','down','left','right']
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
    **Grid:** 2D array of cells with occupant reference and blocking state
    **Rows:** 8
    **Cols:** 8
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
  **Initial Placement:** Airlock tokens placed on predefined start zones; docking stations fixed targets; meteor token pool controlled by AI with a limited duration (temporary block).
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination is adjacent for single-step moves
    - destination is in same row or column for straight-line drag moves and is empty
    - path from source to destination is unobstructed by other tokens, except meteor if applicable
  **Validation Algorithm:**
    **Name:** path_check
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
      - Confirm source contains current player's token
      - If destination is adjacent, ensure empty
      - If destination is along a straight line, ensure all intermediate cells are empty (excluding meteor blocking when applicable)
      - Destination must be a docking-accessible empty cell or a valid path segment
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
  **Result:** No captures occur; only movement and temporary blocking by meteor
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**
      **Affected Count:** 0
    **Parameters:**

    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** count_pieces_to_docking
  **Winner Determination:** Player who docks all airlock tokens first wins; if both achieve docking, winner is the one with fewer moves; otherwise draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceRow
    - sourceCol
    - destRow
    - destCol
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply mechanics.move_validation; update state; apply meteor blocking logic if meteor is deployed; update last_move; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI computes move with minimax; apply mechanics.move_validation and apply meteor deployment; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** MUST define deterministic, replayable rules; UI reads selectors; no gameplay logic duplication in UI components.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple greedy move heuristic.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth minimax search with pruning.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with lookahead and blocking awareness.
**Type:** core


## End Conditions

**Win:**
  **Condition:** All airlock tokens are on their docking stations
  **Check Logic:** Each airlock token must occupy its corresponding docking target; all docking conditions satisfied
  **Priority:** immediate
**Lose:**
  **Condition:** Time limit exceeded or irreversible deadlock
  **Check Logic:** If a player cannot make a legal move within the time window and no progress is possible
  **Priority:** end_turn
**Draw:**
  **Condition:** No legal moves exist for both players for a full turn cycle
  **Check Logic:** Both players have no valid moves within their time window
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click to move a token to an adjacent empty cell
  - Drag to move along straight line to an empty cell
  - Hover for hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser/draw modal
  - Show meteor blocking animation
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


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates accordingly and AI turn starts


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple greedy move within 2s


**Given:** All airlock tokens docked
**When:** Game ends
**Then:** Player wins


**Given:** No legal moves for both players
**When:** Turn passes
**Then:** Draw result is declared


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True