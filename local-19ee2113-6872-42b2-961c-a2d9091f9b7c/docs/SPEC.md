# Portal Tile Swap

## Meta

**Game Name:** Portal Tile Swap
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Players swap adjacent tiles on a 6x6 grid to align matching portals; after each move, the AI inserts blocking tiles to impede future moves.
**Session Minutes:**
  - 5
  - 15

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
    **Grid:** 2D array of portal IDs representing tile types
    **Rows:** 6
    **Cols:** 6
  **Initial State:**
    **Grid:**
      - ['A', 'B', 'C', 'A', 'D', 'B']
      - ['C', 'A', 'D', 'B', 'A', 'C']
      - ['B', 'C', 'A', 'D', 'C', 'A']
      - ['D', 'B', 'C', 'A', 'B', 'D']
      - ['A', 'D', 'B', 'C', 'D', 'A']
      - ['C', 'A', 'D', 'B', 'C', 'A']
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
  **Initial Placement:** Starting grid is predefined for determinism; no randomization on load. AI starts with the standard minimax-based evaluation. Blocking tiles will be inserted after each human move by the AI.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - swap_is_adjacent
    - swap_results_in_valid_alignment_considering_portal_types
  **Validation Algorithm:**
    **Name:** swap_adjacent
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
      **Min Chain Length:** 2
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Verify target cell is within bounds
      - Verify target cell is adjacent to source cell
      - Simulate swap and check if any portal alignment is formed according to game rules
      - Return is_valid and optional preview of resulting grid
**Movement:**
  **Allowed:** swap
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1
**Capture:**
  **Type:** block_placement
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** After a human move, AI places a blocking tile on the board to hinder future moves.
  **Capture Algorithm:**
    **Name:** apply_block_tile
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
        - (None)
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - AI selects an empty cell using its algorithm under current seed
      - Place blocking tile at chosen cell
      - Mark tile as blocking so it cannot be swapped or treated as a normal portal
      - Update board state
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** Player with higher score after no legal moves remain wins; if equal, draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 15
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
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
**Text:** Clear, testable rule with input/output (MUST / SHOULD / MAY) and deterministic outcomes.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use [simple algorithm description].
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use [moderate algorithm description].
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use [advanced algorithm description].
**Type:** core


## End Conditions

**Win:**
  **Condition:** Human has strictly more points and AI has no legal moves
  **Check Logic:** After a turn, evaluate scores; if AI has zero legal moves and human score is greater, human wins
  **Priority:** immediate
**Lose:**
  **Condition:** AI has strictly more points than Human
  **Check Logic:** At evaluation, if ai_score > human_score
  **Priority:** immediate
**Draw:**
  **Condition:** Scores equal and no legal moves for both players
  **Check Logic:** If both players have no legal moves and scores tie
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


**Given:** A valid swap exists per game rules
**When:** Player performs a valid swapping move
**Then:** Board updates by swapping the two tiles; move_count increments; last_move recorded


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm and makes a valid move within 2 seconds


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt a winning or blocking move per its algorithm


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt to block human's winning path


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True