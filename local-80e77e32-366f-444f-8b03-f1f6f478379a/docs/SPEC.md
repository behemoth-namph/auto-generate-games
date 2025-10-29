# Grid Minesweeper

## Meta

**Game Name:** Grid Minesweeper
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Left-click to reveal a cell and right-click to flag suspected mines; reveal all safe cells without detonating a mine to win. The game is deterministic when seeded; mines are placed at initialization using a fixed RNG seed for replayability.
**Session Minutes:**
  - 5
  - 20

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 9
    - 9
  **Neighbors:**
    - N
    - NE
    - E
    - SE
    - S
    - SW
    - W
    - NW
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
    **Grid:** 2D array of cells with fields: mine (bool), revealed (bool), flagged (bool), neighborMineCount (int), exploded (bool)
    **Rows:** 9
    **Cols:** 9
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
    **Last Move:**


## Mechanics

**Setup:**
  **Initial Placement:** Mines are placed on board at initialization using a fixed RNG seed to ensure deterministic replay; board size 9x9 with 10 mines by default.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell is within bounds
    - cell is not already revealed
    - flagging cannot occur on an already revealed cell
    - revealing a flagged cell is allowed only after unflagging
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
      **Min Chain Length:** 0
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Check target within bounds
      - Check cell state is unrevealed for reveal
      - If action is flag, ensure not already flagged or revealed
      - Return validity and a preview of resulting board state
**Movement:**
  **Allowed:** reveal|flag
  **Directions:**
    - (None)
  **Range:** 1
**Capture:**
  **Type:** area_conversion|none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** Revealed cell updates; if zero adjacent mines, flood-fill reveals nearby cells; flagging toggles flag state
  **Capture Algorithm:**
    **Name:** apply_reveal_or_flag
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
      - Reveal: if zero neighboring mines, flood-fill reveal adjacent cells
      - Flag: toggle mine flag on target cell
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** All non-mine cells revealed => human wins; if a mine is detonated => human loses; no draw condition defined unless abrupt stalemate
**Randomness:**
  **Seeded:** True
  **Rng Source:** fixed_seed
  **Notes:** Mine placement is deterministic for replay
**Replay:**
  **Determinism Note:** All random outputs MUST be logged; identical seed yields identical outcomes

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - action
  **Condition:** current_player == 1 AND game.status == playing
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
**Text:** Clear, testable rule with input/output MUST specify error codes for illegal actions.
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
  **Condition:** All non-mine cells revealed
  **Check Logic:** Verify that unrevealed non-mine cells count == 0
  **Priority:** immediate
**Lose:**
  **Condition:** Mine detonated
  **Check Logic:** Game ends immediately when a mine is revealed
  **Priority:** immediate
**Draw:**
  **Condition:** No legal moves exist and not yet won or lost
  **Check Logic:** End-turn when no moves available; declare draw if both players have no legal actions
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to reveal or flag
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser message
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #1e90ff
    **Player 2:** #ff4500
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
**Then:** Captured pieces/cards/territory update exactly as defined


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute winning move


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's winning move


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True