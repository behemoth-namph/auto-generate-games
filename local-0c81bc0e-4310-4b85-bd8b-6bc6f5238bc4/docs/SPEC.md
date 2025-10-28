# Forge Grid Tactics

## Meta

**Game Name:** Forge Grid Tactics
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Players move forge tokens on a 6x6 square grid toward forge targets while avoiding ember blockers. Tokens move to adjacent empty cells; dragging through open lines enables longer moves within a turn. The AI periodically places ember blockers to obstruct paths.
**Session Minutes:**
  - 15
  - 30

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 6
    - 6
  **Neighbors:** up, down, left, right
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #1f1f1f
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
    **Grid:** 2D array of 6x6 cells; each cell either null or an entity id representing a token/blocker
    **Rows:** 6
    **Cols:** 6
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
  **Initial Placement:** Human forge tokens placed on the bottom row (row index 5) columns 0-2. AI forge tokens placed on the top row (row index 0) columns 3-5. Forge targets are designated cells along the top-center area (e.g., row 0, column 2). Ember blockers are introduced by AI during play and are not part of initial layout.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination must be empty
    - destination must be reachable by a valid path according to movement rules
    - path must not pass through blockers or occupied cells
    - movement must adhere to turn movement rules (step or drag within allowed range)
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
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Verify destination is within bounds and empty
      - Compute a contiguous path of empty cells from source to destination
      - Ensure path does not cross ember blockers or other tokens not moved
      - Validate move length against allowed movement mode (step or drag)
**Movement:**
  **Allowed:** step|drag
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** step:1; drag:unlimited within a connected clear path
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture mechanic; blockers persist until removed by special rules or end of game
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
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** First player to occupy a forge target with a token wins. If both players simultaneously occupy forge targets due to a multi-token move, the move that completes the objective first in turn order determines the winner. If no progress occurs and both players have no legal moves for a full round, declare a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 20
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceRow
    - sourceCol
    - destRow
    - destCol
  **Condition:** current_player == 1 AND game.status == playing AND a valid move exists per move_validation
  **Result:** Apply mechanics.move_validation and mechanics.capture; update board and state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update board; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST produce deterministic outcomes given identical inputs.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic that selects the first available legal move.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate search/heuristic strategy with limited lookahead.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning and strategic blocker placement considerations.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_reaches_forge
  **Check Logic:** If any human token occupies a designated forge cell (e.g., row 0, col 2) at end of a turn or during a move that ends on a forge cell
  **Priority:** immediate
**Lose:**
  **Condition:** ai_reaches_forge
  **Check Logic:** If any AI token occupies a forge cell (as defined) at end of a turn
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_or_consecutive_passes
  **Check Logic:** End-of-turn check detects zero legal moves for current player and symmetry for the other player; if true, declare draw
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to perform a move
  - Drag to perform a multi-step move
  - Hover for previews/hints
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
    **Player 1:** #1f1f1f
    **Player 2:** #ffd166
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
**When:** Human performs a valid move
**Then:** Move is applied; board updates; AI turn starts; end conditions checked


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm and makes a valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's potential winning move if applicable


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True