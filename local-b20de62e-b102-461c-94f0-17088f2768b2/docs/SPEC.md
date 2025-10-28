# Token Trail

## Meta

**Game Name:** Token Trail
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A path-building puzzle on a 9x9 grid where players slide same-color tokens into adjacent empty spaces to connect their start area to a goal token; the AI may occasionally place a temporary wall to block paths.
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
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 9x9 grid with cells storing occupantId|null and token color
    **Rows:** 9
    **Cols:** 9
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** int
    **Status:** string
    **Move Count:** int
    **Last Move:** object
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Starting tokens are placed on designated start zones for each color. The AI may intermittently insert a temporary wall token on its turn.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell must be empty
    - movement must be a single orthogonal slide to an adjacent cell
    - selected token color must match current player
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
        - up
        - down
        - left
        - right
      **Min Chain Length:** 1
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - bounds check
      - target cell empty
      - movement is one-step orthogonal
      - token color matches current player
      - update preview if requested
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
  **Result:** no capture occurs
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
  **Method:** path_completion
  **Winner Determination:** A player wins when a connected path exists from their start zone to their goal token; if both sides achieve a path, the player with the longer valid path wins; if neither can progress and both pass, the game is a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - token_id
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
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
**Text:** MUST define clear preconditions, legality, and postconditions for each action; illegal actions MUST yield immediate error {"errorCode", "message"}.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple token-neighborhood heuristic and a shallow search.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate search strategy balancing path building and blocking.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced look-ahead algorithm with seed-based determinism.
**Type:** core


## End Conditions

**Win:**
  **Condition:** path_from_start_to_goal_exists_for_current_player
  **Check Logic:** Check if there's a continuous chain of current player's tokens from their start zone to their goal token
  **Priority:** immediate
**Lose:**
  **Condition:** path_from_start_to_goal_exists_for_opponent
  **Check Logic:** Check if opponent has such a path
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_or_timeout
  **Check Logic:** if both players have no legal moves or both exceed time with no progress
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
**Then:** Expected result with specific values


**Given:** A valid capture exists per game rules
**When:** Player performs a capturing move
**Then:** Captured pieces/territory update exactly as defined


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