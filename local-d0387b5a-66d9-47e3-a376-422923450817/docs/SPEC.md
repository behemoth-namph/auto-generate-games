# Library Card Queue

## Meta

**Game Name:** Library Card Queue
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Click a card to pick, then drag to an adjacent empty space or click a stack to slide one card toward the queue. The AI periodically adds a random librarian block to impede moves.
**Session Minutes:**
  - 5
  - 20

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 5
    - 5
  **Neighbors:** up, down, left, right
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #1e90ff
    **Pieces Played:** int
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** random_block_insertion
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** random_block_insertion
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array of cells; each cell holds occupantId|null and a type (card|librarian|empty)
    **Rows:** 5
    **Cols:** 5
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
  **Initial Placement:** Board initializes with a distribution of card tiles occupying the left side of the board and empty spaces on the right to enable sliding. Librarian blocks start as empty. The queue is conceptually outside the board along the designated exit edge.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - move targets an adjacent empty cell for single-card slide
    - move may instead target a stack of adjacent cards that can slide toward the queue direction
    - blocked by librarian blocks is invalid
  **Validation Algorithm:**
    **Name:** move_validation_logic
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
      **Require Bounded:** false
      **Gravity:** false
    **Steps:**
      - Identify target cell and occupant
      - If target is empty and adjacent, allow slide of single card
      - If target is occupied, check for stack slide toward queue and empty landing space
      - Reject if landing space is blocked by librarian or board boundary
      - Return is_valid and a preview of resulting board state
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
  **Result:** No capture mechanic; blocks only impede or allow slides
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
      - N/A
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** none
**Scoring:**
  **Method:** none
  **Winner Determination:** End conditions determine the winner; no scoring is used

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - targetRow
    - targetCol
  **Condition:** current_player == 1 AND game.status == playing
  **Result:** Apply mechanics.move_validation and mechanics.capture; update board; update last_move; switch to AI; increment move_count; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI selects a legal slide or adjacent-move; applies mechanics.move_validation and mechanics.capture; updates state; switch to human; injects a librarian block every few turns; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST define explicit error handling for illegal actions.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple random or local heuristic to choose a legal slide.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate heuristic considering block positions when selecting slides.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced lookahead to anticipate human moves and block opportunities.
**Type:** core


## End Conditions

**Win:**
  **Condition:** Human cards all successfully moved into the queue
  **Check Logic:** All human-owned card tiles reach the queue position; no legal moves remain for AI that would change this
  **Priority:** immediate
**Lose:**
  **Condition:** AI blocks all paths to queue or no legal moves exist for human
  **Check Logic:** If human has no legal moves on their turn and the queue is not complete
  **Priority:** immediate
**Draw:**
  **Condition:** No legal moves exist for either side on a full turn
  **Check Logic:** End-turn check shows no legal moves for current player and the other side has no immediate winning path
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Queue progress indicator
  - Restart button
**Interactions:**
  - Click to select a card; click a destination empty cell to move
  - Drag to adjacent empty space to slide a card
  - Click a stack to slide toward the queue
  - Click restart to reset the game
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/lose/draw messages
**Board Style:**
  **Cell Size:** 48
  **Grid Line Color:** #333333
  **Disc Colors:**
    **Player 1:** #1e90ff
    **Player 2:** #ffffff
    **Librarian:** #8b5e3c
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
**When:** Player performs a moving action
**Then:** State updates: board occupancy, last_move, and move_count; legal moves recalculated


**Given:** AI turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI selects a valid slide within 2 seconds and updates the board


**Given:** AI can win in one move
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST anticipate and block the human's immediate winning path if feasible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True