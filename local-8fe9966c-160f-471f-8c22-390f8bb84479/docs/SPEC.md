# Reversi (Othello)

## Meta

**Game Name:** Reversi (Othello)
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Players take turns placing discs to sandwich the opponent's discs between their own along straight or diagonal lines; sandwiched discs flip to the current player's color. The game ends when the board is full or no moves remain, and the winner is the player with more discs.
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
    - N
    - NE
    - E
    - SE
    - S
    - SW
    - W
    - NW
  **Id:** board-8x8
  **Coord:**
    - 0
    - 0
  **Tags:**
    - board
    - reversi
    - grid
    - 8x8
  **Occupantid:** None
  **Visibility:** public
  **Rows:** 8
  **Cols:** 8
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** black
    **Pieces Played:** 0
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Id:** 2
    **Owner:** 2
    **Type:** ai
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Color:** white
  **Initial State:**
    **Id:** 2
    **Owner:** 2
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 8x8 grid with 64 cells; each cell can be empty or contain a disc of a color
    **Rows:** 8
    **Cols:** 8
  **Initial State:**
    **Cells:**
      - [0, 0, 0, 0, 0, 0, 0, 0]
      - [0, 0, 0, 0, 0, 0, 0, 0]
      - [0, 0, 0, 0, 0, 0, 0, 0]
      - [0, 0, 0, 1, 2, 0, 0, 0]
      - [0, 0, 0, 2, 1, 0, 0, 0]
      - [0, 0, 0, 0, 0, 0, 0, 0]
      - [0, 0, 0, 0, 0, 0, 0, 0]
      - [0, 0, 0, 0, 0, 0, 0, 0]
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
  **Initial Placement:** Place four discs at the center in a diagonal pattern: positions (3,3)=Black, (3,4)=White, (4,3)=White, (4,4)=Black.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - creates at least one sandwich of opponent discs in any direction
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
        - N
        - NE
        - E
        - SE
        - S
        - SW
        - W
        - NW
      **Min Chain Length:** 1
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Ensure target cell is empty
      - For each of 8 directions, scan contiguous opponent discs followed by a current-player disc
      - If at least one direction yields a sandwich, mark move valid and capture preview
**Movement:**
  **Allowed:** placement
  **Directions:**
    - N
    - NE
    - E
    - SE
    - S
    - SW
    - W
    - NW
  **Range:** 1
**Capture:**
  **Type:** line_flipping
  **Directions:**
    - N
    - NE
    - E
    - SE
    - S
    - SW
    - W
    - NW
  **Require Sandwich:** True
  **Chain Capture:** True
  **Min Chain Length:** 1
  **Result:** All opponent discs sandwiched along valid directions are flipped to current player's color.
  **Capture Algorithm:**
    **Name:** apply_line_flips
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
        - N
        - NE
        - E
        - SE
        - S
        - SW
        - W
        - NW
      **Min Chain Length:** 1
      **Require Bounded:** True
    **Steps:**
      - For each direction, walk from placed disc outward
      - If a consecutive run of opponent discs ends with a current-player disc, flip all in that run
      - Repeat for all directions
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** count_pieces
  **Winner Determination:** At end of game, compare counts; higher count wins; if equal, draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Ai Move Time Limit Ms:** 2000
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
  **Timeout Ms:** 2000
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output (use MUST/SHOULD/CAN)
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple evaluation heuristic and return a valid move within the time limit.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth search with pruning strategy.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced evaluation and look-ahead strategy to prioritize flips and stability.
**Type:** core


## End Conditions

**Win:**
  **Condition:** player1_pieces > player2_pieces
  **Check Logic:** Count discs at end of game; determine winner
  **Priority:** end_turn
**Lose:**
  **Condition:** player1_pieces < player2_pieces
  **Check Logic:** Count discs at end of game; determine loser
  **Priority:** end_turn
**Draw:**
  **Condition:** player1_pieces == player2_pieces
  **Check Logic:** Count discs at end of game; determine draw
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
**Then:** Move is legal only if it flips at least one disc; state updates accordingly


**Given:** A valid capture exists per game rules
**When:** Player performs a capturing move
**Then:** Captured discs are flipped exactly as defined by capture rules


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic and makes a valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST attempt to select a move that leads to a winning end state if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block or choose the best defensive move when appropriate


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Ai Move Time Limit Ms:** 2000
**Seed:** 123456789