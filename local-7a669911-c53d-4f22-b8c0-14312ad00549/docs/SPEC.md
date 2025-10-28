# Connect Four

## Meta

**Game Name:** Connect Four
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Drop tokens into a vertical 7x6 grid by selecting a column; tokens fall to the lowest available row. The goal is to connect four tokens in a row horizontally, vertically, or diagonally before the AI does.
**Session Minutes:**
  - 5
  - 15

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
    - up-left
    - up-right
    - down-left
    - down-right
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #e74c3c
    **Pieces Played:** 0
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Id:** 2
    **Type:** ai
    **Player Number:** 2
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Id:** 2
    **Type:** ai
    **Player Number:** 2
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array with dimensions 6x7; values: 0 = empty, 1 = human, 2 = AI
    **Rows:** 6
    **Cols:** 7
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
  **Initial Placement:** All cells start empty; no pre-filled tokens.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - selected column is not full
    - landing row is computed by gravity (token occupies lowest available cell)
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
      **Gravity:** True
    **Steps:**
      - If the column's top cell is non-empty, return invalid.
      - Compute landing_row as the lowest empty row in the selected column.
      - Preview the board state with current player's token placed at (landing_row, col).
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
  **Result:** No piece removal or flipping; win condition checked separately via four-in-a-row.
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**

    **Parameters:**

    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
**Scoring:**
  **Method:** points
  **Winner Determination:** The winner is the first player to form a four-in-a-row; if the board fills without any four-in-a-row, the game is a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - columnIndex
  **Condition:** current_player == 1 AND game.status == playing AND columnIndex is a valid move
  **Result:** Apply mechanics.move_validation; apply mechanics.capture (if any); update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using minimax; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
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
**Text:** AI with difficulty 'easy' MUST use [simple algorithm description].
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use minimax with depth 4 and simple pruning.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an enhanced minimax with depth 6 and heuristic evaluation.
**Type:** core


## End Conditions

**Win:**
  **Condition:** current_player achieved four-in-a-row
  **Check Logic:** Scan board after every move for four consecutive tokens (horizontal, vertical, diagonal)
  **Priority:** immediate
**Lose:**
  **Condition:** opponent achieved four-in-a-row
  **Check Logic:** Equivalent to opponent's win condition detected after their move
  **Priority:** immediate
**Draw:**
  **Condition:** board is full and no four-in-a-row exists
  **Check Logic:** No empty cells remain and no winner detected
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
  - Display winner/loser/draw message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #e74c3c
    **Player 2:** #f1c40f
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
**When:** Player performs a valid drop in a column
**Then:** Board updates with human token in the lowest available row in that column


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple column-first heuristic and makes a valid move within 2s


**Given:** AI can win in one move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's imminent winning move if possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True