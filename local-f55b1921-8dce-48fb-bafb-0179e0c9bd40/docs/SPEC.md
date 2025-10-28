# Color Cascade Grid

## Meta

**Game Name:** Color Cascade Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Swap adjacent tokens on a 7x7 grid to form color streaks in rows or columns; streaks clear and colors cascade to fill gaps. The AI introduces color goblins that reseed colors to complicate future moves.
**Session Minutes:**
  - 5
  - 60

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
    **Grid:** 2D array of 7x7 cells; each cell stores a color_id or null for empty
    **Rows:** 7
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
  **Initial Placement:** 7x7 grid seeded with colored tokens; colors drawn from a fixed palette; board starts fully occupied with no input from UI.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** False
  **Validity Checks:**
    - target must be adjacent to the selected token
    - swap must produce at least one horizontal or vertical color streak of length >= 3
    - resulting state must be deterministic and replayable
  **Validation Algorithm:**
    **Name:** swap_and_pattern_check
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
      **Min Chain Length:** 3
      **Require Bounded:** true
      **Gravity:** true
    **Steps:**
      - Ensure a token is selected by current player
      - Check target cell is within 1-step Manhattan distance and is a neighbor
      - Simulate swapping the two tokens
      - Scan rows and columns for any streaks of a single color with length >= 3
      - If at least one streak exists, mark as valid and provide affected area as preview
      - If no streaks, mark as invalid with appropriate error
**Movement:**
  **Allowed:** swap
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1
**Capture:**
  **Type:** piece_removal
  **Directions:**
    - horizontal
    - vertical
  **Require Sandwich:** false
  **Chain Capture:** true
  **Min Chain Length:** 3
  **Result:** All cells forming a streak of length >= 3 in any row or column are removed (set to empty). Subsequent cascades apply gravity to fill gaps and colors may reseed via goblins.
  **Capture Algorithm:**
    **Name:** remove_streaks_and_cascade
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
        - horizontal
        - vertical
      **Min Chain Length:** 3
      **Gravity:** true
    **Steps:**
      - Identify all horizontal and vertical streaks of length >= min_chain_length
      - Remove all cells in identified streaks (set to empty)
      - Apply gravity to collapse tokens downward if gravity is enabled
      - If goblins trigger reseed, recolor a subset of empty cells with new colors
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** count_pieces
  **Winner Determination:** At end of game, the player with more tokens on the board is the winner; if equal, it's a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - fromRow
    - fromCol
    - toRow
    - toCol
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; update last_move; switch to AI; check end conditions
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
**Text:** Clear, testable rule with input/output MUST define a deterministic outcome for every valid input; illegal actions MUST yield an error with { errorCode, message }.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple swap-evaluation heuristic.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate minimax with limited depth and pruning.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search (e.g., MCTS or full-depth minimax with heuristics).
**Type:** core


## End Conditions

**Win:**
  **Condition:** player_tokens_count == 0 AND ai_tokens_count > 0
  **Check Logic:** Count tokens of each color on the board; if all AI color tokens remain and Player color tokens are zero, AI loses; reported as player win.
  **Priority:** immediate
**Lose:**
  **Condition:** ai_tokens_count == 0 AND player_tokens_count > 0
  **Check Logic:** Count tokens of each color on the board; if all Player color tokens remain and AI color tokens are zero, AI wins.
  **Priority:** immediate
**Draw:**
  **Condition:** no legal moves exist for both players
  **Check Logic:** Deterministic legal-move scan; if zero moves for both sides, declare draw.
  **Priority:** end_turn
  **Condition:** token counts equal and no further captures possible
  **Check Logic:** Compare counts; if equal and no moves/captures left, declare draw.
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
    **Player 2:** #ff8c00
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
**Then:** Captured pieces removed and board cascades deterministically


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST avoid blocking human's obvious winning move if rule requires it (deterministic play)


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True