# Fifteen Tile Puzzle

## Meta

**Game Name:** Fifteen Tile Puzzle
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A deterministic 15-puzzle on a 4x4 grid. The human and AI take turns sliding a tile adjacent to the empty space into the gap, with the goal of arranging tiles 1-15 in order and the empty space at the end. The UI renders on HTML5 Canvas and all logic is deterministic.
**Session Minutes:**
  - 5
  - 15

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 4
    - 4
  **Neighbors:**
    - up
    - down
    - left
    - right
  **Tiles:**
    - [1, 2, 3, 4]
    - [5, 6, 7, 8]
    - [9, 10, 11, 12]
    - [13, 14, 15, 0]
  **Empty Position:**
    **Row:** 3
    **Col:** 3
  **Rows:** 4
  **Cols:** 4
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
    **Algorithm:** random
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 200
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** random
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 200
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Tiles:**
      - [1, 2, 3, 4]
      - [5, 6, 7, 8]
      - [9, 10, 11, 12]
      - [13, 14, 15, 0]
    **Empty Position:**
      **Row:** 3
      **Col:** 3
    **Rows:** 4
    **Cols:** 4
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:**
      **Type:** integer
      **Allowed Values:**
        - 1
        - 2
    **Status:** string (playing|human_wins|ai_wins|draw)
    **Move Count:** int
    **Last Move:** object
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Solvable scrambled 4x4 grid with tiles 1-15 and one empty space; the initial permutation must be solvable.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - tile adjacent orthogonally to the empty space
    - tile value is not 0 (empty)
    - parity check preserves puzzle solvability
  **Validation Algorithm:**
    **Name:** adjacency_check
    **Inputs:**
      - tileRow
      - tileCol
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
      - Identify empty cell coordinates
      - Verify tile is orthogonally adjacent to empty cell
      - Swap tile with empty cell if valid
      - Update parity and board state
      - Return updated board as preview
**Movement:**
  **Allowed:** slide
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
  **Result:** N/A for sliding puzzle; no captures occur
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - tileRow
      - tileCol
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
      - N/A
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** none
  **Winner Determination:** End state is solved when tiles are in order with 0 at the end

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - tileRow
    - tileCol
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply move_validation and movement; update board; set last_move with player_id 1; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using algorithm; apply move_validation and movement; update board; set last_move with player_id 2; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output (MUST/SHOULD/CAN)
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use simple adjacent move or random valid move strategy.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a heuristic-guided search toward the solved state.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning and deterministic evaluation.
**Type:** core


## End Conditions

**Win:**
  **Condition:** board_solved
  **Check Logic:** Board matches the goal configuration (tiles in order with 0 at the end)
  **Priority:** immediate
**Lose:**
  **Condition:** board_solved_by_ai
  **Check Logic:** Board matches the goal configuration and last_move by AI
  **Priority:** end_turn
**Draw:**
  **Condition:** no legal moves exist and board not solved
  **Check Logic:** Legal moves set is empty while board is not solved
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to propose a move (tile adjacent to empty space)
  - Drag toward the gap to slide (where supported)
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner message
**Board Style:**
  **Cell Size:** 64
  **Grid Line Color:** #333333
  **Tile Colors:**
    **Default:** #2a64c7
    **Numbers:** #ffffff
    **Empty:** #00000000
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
**Then:** Board updates exactly with the swapped tile into the empty space; move_count increments; last_move records Player 1


**Given:** AI turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI selects a valid adjacent move within 2 seconds and updates the board accordingly


**Given:** AI can solve in one move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move when available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST not block human's direct winning path if deterministic constraints allow


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True