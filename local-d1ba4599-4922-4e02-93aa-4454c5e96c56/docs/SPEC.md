# Lantern Link

## Meta

**Game Name:** Lantern Link
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** On an 8x8 grid, players slide lantern tokens along rows or columns to connect them; click to pick a lantern, then click an unobstructed line to move it tile-by-tile, or drag along a straight path. An AI introduces light-blockers to complicate routes. The game is deterministic, replayable, and UI-decoupled.
**Session Minutes:**
  - 5
  - 60

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 8
    - 8
  **Neighbors:**
    - N
    - S
    - E
    - W
  **Cell Schema:**
    **Id:** string
    **Coord:**
      - 0
      - 0
    **Tags:**
      - (None)
    **Occupantid:** None
    **Visibility:** public
  **Cells Metadata:**
    **Coord:**
      - 0
      - 0
    **Id:** cell_0_0
    **Tags:**
      - start
    **Occupantid:** None
    **Visibility:** public
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #1f7a1f
    **Pieces Played:** 0
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** minimax
    **Difficulty:** hard
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** hard
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 8x8 lattice of cells with occupancy states
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
  **Initial Placement:** A symmetric starter layout places a subset of lantern tokens for the human player on accessible cells along the front rows, with empty spaces enabling initial slides. AI lights are not blocking at setup but will be introduced progressively by the AI on its turns.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - move originates from a lantern owned by the current player
    - destination is empty
    - movement is along a single row or a single column
    - path between origin and destination is clear of blockers and other lanterns
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
        - north
        - south
        - east
        - west
      **Min Chain Length:** 1
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Verify a lantern belonging to current_player exists at (row, col).
      - Determine direction and trace a straight line to the target cell.
      - Ensure all intermediate cells are empty and within board bounds.
      - Destination must be empty; if valid, return new coordinates as preview.
      - Return is_valid = true with preview of resulting board state.
**Movement:**
  **Allowed:** slide
  **Directions:**
    - north
    - south
    - east
    - west
  **Range:** unbounded
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No captures occur in Lantern Link; goal is connectivity of lanterns via valid slides
  **Capture Algorithm:**
    **Name:** none
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
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** At end, the player with more connected lanterns (via valid slides) wins; equal counts result in a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - direction
    - destination
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
**Text:** All gameplay must be deterministic and replayable given the same seed and inputs. This game does not use nondeterministic randomness in core logic.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic-based slide generation.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth search suitable for turn-based pathfinding.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning and deterministic evaluation.
**Type:** core


## End Conditions

**Win:**
  **Condition:** Human lanterns connected
  **Check Logic:** All human lanterns are connected through a chain of valid slides with no blockers interrupting the chain
  **Priority:** immediate
**Lose:**
  **Condition:** AI blocks all human moves
  **Check Logic:** On human turn, there are zero legal moves available for the human player
  **Priority:** end_turn
**Draw:**
  **Condition:** No legal moves for either side for a full round
  **Check Logic:** End of a complete cycle with no legal moves detected for both players
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
    **Player 1:** #1f7a1f
    **Player 2:** #1b4d90
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
**When:** Player performs a valid sliding move
**Then:** State updates reflect the slide, and last_move is recorded


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic, makes a valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt to block human's potential winning move


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True