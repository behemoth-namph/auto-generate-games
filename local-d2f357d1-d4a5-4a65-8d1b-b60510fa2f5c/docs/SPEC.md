# Bubble Grid Matcher

## Meta

**Game Name:** Bubble Grid Matcher
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Shoot bubbles from the top toward a hex-grid cluster; connect three or more of the same color to pop; popped bubbles leave gaps that cause cascades downward; play alternates between a human and an AI.
**Session Minutes:**
  - 5
  - 15

## State

**Board:**
  **Type:** grid
  **Topology:** hex
  **Dimensions:**
    - 12
    - 8
  **Neighbors:** N, NE, SE, S, SW, NW
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #1e90ff
    **Pieces Played:** 0
    **Score:** 0
  **Initial State:**
    **Pieces Played:** 0
    **Score:** 0
  **Name:** AI
  **Properties:**
    **Id:** 2
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Score:** 0
    **Max Think Time Ms:** 2000
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Max Think Time Ms:** 2000
  **Name:** Board
  **Properties:**
    **Grid:** Hexagonal offset grid with 12 rows x 8 columns; each cell holds a color bubble or is empty
    **Rows:** 12
    **Cols:** 8
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
**Roster:**
  **Id:** 1
  **Role:** human
  **Id:** 2
  **Role:** ai
**Seed:** 123456789

## Mechanics

**Setup:**
  **Initial Placement:** Starting bubbles are pre-populated in the upper region to form initial clusters; subsequent bubbles spawn from the top row for the human player's shots.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell is empty
    - target is within allowable spawn zone (top region)
    - color is valid and within palette
    - placement does not violate grid boundaries
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
        - SE
        - S
        - SW
        - NW
      **Min Chain Length:** 3
      **Require Bounded:** False
      **Gravity:** True
    **Steps:**
      - Locate target cell; verify empty
      - Compute connected group of same-color bubbles from target cell
      - If group size >= min_chain_length, mark as captured
      - Apply capture then gravity
      - If no capture, place bubble and proceed
**Movement:**
  **Allowed:** placement
  **Directions:**
    - top_insertion
  **Range:** any
**Capture:**
  **Type:** piece_removal
  **Directions:**
    - N
    - NE
    - SE
    - S
    - SW
    - NW
  **Require Sandwich:** False
  **Chain Capture:** True
  **Min Chain Length:** 3
  **Result:** Any connected group of 3+ same-color bubbles is removed; gravity causes bubbles above to fall; new bubbles may create cascades
  **Capture Algorithm:**
    **Name:** remove_captured
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
        - N
        - NE
        - SE
        - S
        - SW
        - NW
      **Min Chain Length:** 3
      **Require Bounded:** False
    **Steps:**
      - Identify connected group from placed cell
      - If size >= min_chain_length -> remove group
      - Apply gravity to re-stack bubbles
      - Repeat cascades until stable
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** Human and AI scores are compared; winner is the player with the higher score when the end condition is reached; if a board-clearing end is triggered, higher score wins.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
    - color
  **Condition:** current_player == 1 AND game.status == playing AND target cell is empty AND move is valid
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
**Text:** CLEAR, testable rule with input/output MUST / SHOULD / MAY. All actions MUST be deterministic.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within max_think_time_ms of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple evaluation strategy.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth search and heuristic evaluation.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search with deeper horizon and pruning.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_score_target_reached OR board_cleared
  **Check Logic:** Human reaches target score or board is cleared
  **Priority:** immediate
**Lose:**
  **Condition:** ai_score_target_reached
  **Check Logic:** AI reaches target score
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_players
  **Check Logic:** End turn when both players have no legal moves
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
  **Target Fps:** 60
  **Max Load Time Ms:** 1000
  **Max Ai Think Time Ms:** 2000

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid capture exists per game rules
**When:** Player performs a capturing move
**Then:** Captured bubbles removed and board cascades accordingly


**Given:** AI's turn, difficulty='medium'
**When:** AI calculates move
**Then:** AI uses minimax with depth 4 within 2000 ms and makes valid move


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST execute winning move


**Given:** Human can win next turn
**When:** AI's turn, difficulty='hard'
**Then:** AI MUST block human's winning move


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Ai Max Think Time Ms:** 2000