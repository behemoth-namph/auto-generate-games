# Garden Grid Shuffle

## Meta

**Game Name:** Garden Grid Shuffle
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Tap a flower token and move it to a matching neighboring cell to group and clear connected same-color flowers. The AI occasionally sprouts weed tokens that must be avoided.
**Session Minutes:**
  - 5
  - 15

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
    **Color:** #2e8b57
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
    **Grid:** 7x7 grid with initial flower tokens distribution; internal representation as 2D array of cell objects
    **Rows:** 7
    **Cols:** 7
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
  **Initial Placement:** Flowers of various colors populate the 7x7 grid with empty cells distributed to allow initial moves; no weeds at start.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell is empty
    - target is adjacent to the chosen token
    - target color matches the movable token
  **Validation Algorithm:**
    **Name:** grid_path_check
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
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Verify target is empty
      - Verify there exists a selected token owned by current_player
      - Verify target is adjacent and color matches
      - Apply grouping and clearing if a connected region of same-color tokens forms
      - Update board and score accordingly
**Movement:**
  **Allowed:** tap_move
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1
**Capture:**
  **Type:** area_conversion
  **Directions:**
    - up
    - down
    - left
    - right
  **Require Sandwich:** False
  **Chain Capture:** True
  **Min Chain Length:** 2
  **Result:** A connected cluster of same-color flowers is cleared, awarding points; any weed tokens newly spawned remain
  **Capture Algorithm:**
    **Name:** apply_area_clear
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
        - up
        - down
        - left
        - right
      **Min Chain Length:** 2
      **Require Bounded:** False
    **Steps:**
      - Compute connected component of same-color tokens containing the moved token
      - If size >= 2, remove them and update current_player score
      - Trigger weed spawn with AI-spawn rule (non-deterministic? but deterministically seeded)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** none
**Scoring:**
  **Method:** points
  **Winner Determination:** Most points at end of game or clear all tokens; in tie, draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - tokenId
    - targetRow
    - targetCol
  **Condition:** current_player == 1 AND game.status == playing AND isMoveLegal(tokenId, targetRow, targetCol)
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; switch to AI; check end conditions
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
**Text:** Clear, testable rule with input/output MUST define moves deterministically; illegal actions yield an error.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple random-adjacent-move algorithm.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth minimax with pruning.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced predictive search with transposition table hints.
**Type:** core


## End Conditions

**Win:**
  **Condition:** player_points_win
  **Check Logic:** player reaches predetermined points or triggers ending condition
  **Priority:** end_turn
**Lose:**
  **Condition:** ai_points_win
  **Check Logic:** AI reaches predetermined points or triggers ending condition
  **Priority:** end_turn
**Draw:**
  **Condition:** no_legal_moves_remaining
  **Check Logic:** no moves exist for both players
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board area with 7x7 grid
  - Current player indicator (You vs AI)
  - Scores and remaining tokens
  - AI thinking indicator
  - Status and end-of-game modal
  - Restart button
  - Accessibility annotations
**Interactions:**
  - Tap to select a token
  - Tap adjacent empty or matching cell to move
  - Hover for previews
  - Click restart
**Feedback:**
  - Highlight selected token
  - Show valid moves
  - Animate clearings
  - Show AI moves and weeds spawns
  - Display winner/loser message
**Board Style:**
  **Cell Size:** 54
  **Grid Line Color:** #2d2d2d
  **Disc Colors:**
    **Player 1:** #2e8b57
    **Player 2:** #d2691e
    **Outline:** #5a9e58
  **Valid Move Highlight:**
    **Enabled:** True
    **Color:** #66cc66
  **Last Move Highlight:**
    **Enabled:** True
    **Color:** #ffd700
  **Flip Animation:**
    **Enabled:** True
    **Duration Ms:** 200

## Technical

**Platform:** HTML5 Canvas
**Performance:**
  **Target Fps:** 60
  **Max Load Time Ms:** 1000
  **Max Ai Think Time Ms:** 2000

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid move exists
**When:** Player makes a legal move
**Then:** State updates accordingly and pieces may clear


**Given:** AI's turn
**When:** AI calculates move
**Then:** AI uses minimax-based algorithm within time bound and makes a valid move


**Given:** AI can win in one move
**When:** AI's turn
**Then:** AI must take winning move if available


**Given:** Human can win next turn
**When:** AI's turn
**Then:** AI must block human's potential winning move if possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True