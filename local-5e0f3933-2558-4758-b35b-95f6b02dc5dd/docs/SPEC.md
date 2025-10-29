# Connect Four Solo

## Meta

**Game Name:** Connect Four Solo
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Drop discs into a 7x6 grid by column; gravity places the piece in the lowest available cell. First to connect four in a row, column, or diagonal wins; playing against a basic AI.
**Session Minutes:**
  - 5
  - 15
**Seed:** ABC123

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 6
    - 7
  **Neighbors:**
    - up
    - down
    - left
    - right
    - diag_up_left
    - diag_up_right
    - diag_down_left
    - diag_down_right
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
    **Id:** 2
    **Type:** ai
    **Color:** #ffffff
    **Pieces Played:** 0
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:**
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
    **Rows:** 6
    **Cols:** 7
  **Initial State:**
    **Grid:**
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None]
    **Rows:** 6
    **Cols:** 7
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
  **Initial Placement:** All discs start off-board; each move drops a disc into a chosen column and gravity places it in the lowest available row.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - column_has_space
    - column_index_in_range
    - board_not_full_in_column
  **Validation Algorithm:**
    **Name:** drop_gravity
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
        - down
      **Min Chain Length:** 4
      **Require Bounded:** False
      **Gravity:** True
    **Steps:**
      - If selected column is full, return invalid.
      - Find the lowest empty row in the column.
      - Return preview position for placement.
      - Apply placement if valid.
**Movement:**
  **Allowed:** placement
  **Directions:**
    - down
  **Range:** 1
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture mechanics in Connect Four; only placement and win condition evaluation.
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
  **Pass If No Valid Move:** False
  **Extra Turn Conditions:** none
**Scoring:**
  **Method:** points
  **Winner Determination:** A player wins when they achieve four in a row; if the board fills with no winner, the game is a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - columnIndex
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
**Text:** Clear, testable rule with input/output (MUST/SHOULD/CAN).
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple algorithm description to pick a valid column.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate algorithm description with limited lookahead.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an enhanced algorithm description with planning and pruning.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_win_by_connect_four
  **Check Logic:** Detect four consecutive human discs in any direction
  **Priority:** immediate
  **Condition:** ai_win_by_connect_four
  **Check Logic:** Detect four consecutive AI discs in any direction
  **Priority:** immediate
**Lose:**
  **Condition:** human_lost_by_ai_four
  **Check Logic:** Mirror of human win condition
  **Priority:** immediate
**Draw:**
  **Condition:** board_full_with_no_winner
  **Check Logic:** All cells occupied and no four-in-a-row
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap a column to drop a disc
  - Hover for column preview
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser/draw message
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


**Given:** A valid placement exists per game rules
**When:** Player places in a valid column
**Then:** Board updated; turn passes to AI; last_move recorded


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI selects a valid column within 2 seconds and updates state


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's potential winning move if possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Rng Seed:** ABC123
**Seed Source:** meta.seed