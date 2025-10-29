# Solitaire Studio

## Meta

**Game Name:** Solitaire Studio
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A compact Klondike-style solitaire on a grid where the player moves exposed tableau cards and stock/waste to build four foundations in order; AI turns operate on a fixed-seed, deterministic strategy.
**Session Minutes:**
  - 10
  - 60

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 7
    - 9
  **Neighbors:**
    - up
    - down
    - left
    - right
**Allegiance:** none
**Holdings Present:** True
**Board Present:** True
**Physics Context:** static
**Turn Mode:** sequential
**Randomness:**
  **Type:** deterministic
  **Seed Source:** server_seed
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** green
    **Pieces Played:** 0
    **Owner Id:** 1
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** rule_based
    **Difficulty:** medium
    **Depth:** 0
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Id:** 2
    **Owner Id:** 2
    **Type:** ai
  **Initial State:**
    **Algorithm:** rule_based
    **Difficulty:** medium
    **Depth:** 0
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Id:** 2
    **Owner Id:** 2
    **Type:** ai
  **Name:** Board
  **Properties:**
    **Grid:** rectangular 7x9 representation containing foundation slots, stock, waste, and 7 tableau piles as stacked piles
    **Rows:** 7
    **Cols:** 9
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
  **Initial Placement:** Klondike-style: 7 tableau piles with incremental face-down cards and a face-up top card; 24-card stock; waste starts empty; foundations empty.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - moves must place onto foundations or properly exposed tableau cards
    - no invalid rank/colour jumps
    - only face-up sequences may be moved as a unit
    - stock to waste is allowed when tapping stock
    - foundations must follow Ace to King in matching suit
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
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Identify source card(s) to move
      - Validate selection is a legal sequence
      - Check destination legality (foundation or tableau stacking rules)
      - Ensure moves do not violate suit/rank sequence
      - Produce a preview of resulting layout
**Movement:**
  **Allowed:** slide
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** any
**Capture:**
  **Type:** card_removal
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** When a move is executed, source cards are removed from their previous stack or moved to the destination; foundations accumulate cards; tableau cards flip face-down if uncovered
  **Capture Algorithm:**
    **Name:** apply_move
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
      - Validate move via move_validation
      - Apply changes to board state
      - Flip face-down cards if needed
      - Update foundations and tableau counts
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** When all foundations reach King, human wins; if no moves and stock depleted, draw or loss as defined

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - source
    - destination
  **Condition:** current_player == 1 AND game.status == playing AND move is valid
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
**Text:** AI with difficulty 'easy' MUST use a simple heuristic that chooses the first legal move.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a rule-based strategy prioritizing foundational moves and exposing new tableau cards.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with pruning to maximize foundation progress within depth limits.
**Type:** core


## End Conditions

**Win:**
  **Condition:** All foundations complete
  **Check Logic:** Each foundation reaches King of its suit in order
  **Priority:** immediate
**Lose:**
  **Condition:** No moves left and stock depleted
  **Check Logic:** No legal moves exist and stock is empty
  **Priority:** end_turn
**Draw:**
  **Condition:** No legal moves and no possibility to progress within given seed
  **Check Logic:** Stalemate state with fixed seed
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board area (tableau, stock, waste, foundations)
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Status banner / message
  - Restart button
**Interactions:**
  - Click/tap to select card(s) and destination
  - Drag-and-drop or click-to-move sequences
  - Hover for hints
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
    **Player 1:** #0a0a0a
    **Player 2:** #f0f0f0
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


**Given:** A valid move exists per Klondike rules
**When:** Player performs a valid move
**Then:** State updates accordingly and logs move


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, moves within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST execute a potential winning move


**Given:** Human can win next turn
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST block or prolong to prevent immediate win


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True