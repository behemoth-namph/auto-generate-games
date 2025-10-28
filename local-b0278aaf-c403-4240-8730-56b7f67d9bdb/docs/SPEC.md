# Stack Swap Mosaic

## Meta

**Game Name:** Stack Swap Mosaic
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Two adjacent tokens swap on a 6x6 grid; matching colors in horizontal or vertical lines of three or more disappear, gravity fills gaps, and new tokens drop from the top. The player competes against an AI to reach a target score within a fixed number of moves in a deterministic, replayable puzzle loop.
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
    **Owner Id:** 2
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 6x6 array of tokens; each cell holds a color value or null for empty
    **Rows:** 6
    **Cols:** 6
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Target Score:** 1000
    **Max Moves:** 20
    **Last Move:** None
    **Human Score:** 0
    **Ai Score:** 0
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Target Score:** 1000
    **Max Moves:** 20
    **Last Move:** None
    **Human Score:** 0
    **Ai Score:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Deterministic seed-based randomization to generate a 6x6 grid of colored tokens; initial layout may cascade depending on seed.
  **Starting Player Rule:** human
  **Seed:** 12345
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - swap must be between adjacent cells
    - swap must result in at least one line of 3+ after resolution (or be rejected as invalid)
  **Validation Algorithm:**
    **Name:** swap_and_resolve
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
      **Gravity:** True
      **Resolve Chain Reactions:** True
    **Steps:**
      - Identify adjacent swap pair
      - Execute swap on a copy of the board
      - Apply line-match removal for all matches length >= 3 in rows and columns
      - Apply gravity to drop tokens and fill from the top
      - Repeat cascade detection and removal until no new matches exist
      - Compute score delta from removed tokens
      - Return is_valid and resulting board view
**Movement:**
  **Allowed:** swap
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** 1
**Capture:**
  **Type:** line_removal
  **Directions:**
    - horizontal
    - vertical
  **Require Sandwich:** False
  **Chain Capture:** True
  **Min Chain Length:** 3
  **Result:** All horizontal or vertical lines of 3+ matching colors disappear; gravity pulls tokens down and new tokens fill from the top; score increases by number of tokens removed; cascades continue until stable
  **Capture Algorithm:**
    **Name:** apply_line_matches_and_gravity
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Removed Count:** int
      **New Board:** object
    **Parameters:**
      **Directions:**
        - horizontal
        - vertical
      **Min Chain Length:** 3
      **Chain Reaction:** True
    **Steps:**
      - Scan for horizontal and vertical runs of length >= 3
      - Remove matched tokens (set to null) and update score by count
      - Apply gravity to collapse columns
      - Generate new tokens to fill vacated cells at the top
      - Repeat until no new matches appear
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after max_moves or when target_score is reached
**Scoring:**
  **Method:** points
  **Description:** Points awarded for each removed token; possible multipliers for cascading clears; score tracked per player
  **Winner Determination:** First to reach target_score wins; if both fail to reach target within max_moves, determine winner by higher score; draw if equal

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 2
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - direction
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply mechanics.move_validation and mechanics.capture; update board, score, and move_count; switch to AI; check end conditions
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
**Text:** Core determinism MUST hold: all game logic is deterministic given a fixed seed; replays MUST reproduce identical outcomes.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple random-adjacent-swap heuristic.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a lookahead to avoid obviously bad moves.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use a strategic search with pruning and evaluation tuning.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_score >= target_score
  **Check Logic:** If human reaches target_score, human wins; priority immediate
  **Priority:** immediate
**Lose:**
  **Condition:** ai_score >= target_score
  **Check Logic:** If AI reaches target_score, AI wins (human loses); priority immediate
  **Priority:** immediate
**Draw:**
  **Condition:** move_count >= max_moves AND (human_score < target_score) AND (ai_score < target_score)
  **Check Logic:** No moves remaining and neither reached target
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
  - Score display
  - Target score
  - Moves remaining
**Interactions:**
  - Click/tap two adjacent cells to propose a swap
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/lose/draw message
  - Cascade animation effects
**Board Style:**
  **Cell Size:** 45
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #1e90ff
    **Player 2:** #ff6347
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
  **Target Fps:** 60
  **Max Load Time Ms:** 1000
  **Max Ai Think Time Ms:** 2000

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid swap exists per game rules
**When:** Player performs a valid swap
**Then:** Board updates, matches resolved, score updated accordingly


**Given:** AI turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** Human reaches target score on their turn
**When:** Score update occurs
**Then:** Game ends with human as winner


**Given:** Both players exhaust moves without reaching target
**When:** max_moves reached
**Then:** Draw condition triggered


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Ai Time Budget By Difficulty:**
  **Easy:** 1000
  **Medium:** 1500
  **Hard:** 2000