# Orb Collector Grid

## Meta

**Game Name:** Orb Collector Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Collect orbs by sliding them on a 7x7 grid to reach goal sockets; players alternate moves. AIs introduces shadow orbs that shift available spaces, influencing future options. Movement is either to an adjacent empty cell or sliding along a straight line to a further empty cell.
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
  **Neighbors:** ["up","down","left","right"]
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #000000
    **Pieces Played:** int
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
    **Grid:** 2D array representation of 7x7 cells; cell values: 0=empty, 1=human orb, 2=AI orb, 3=goal_socket, 4=shadow_orb
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
  **Initial Placement:** Each player starts with a set of orbs on their home rows; goal sockets are placed on designated sockets; the center region remains open as the playfield; empty spaces exist between placements.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - Target cell must be adjacent empty cell or reachable via a straight-line slide with a clear path.
    - Path between start and target must be unblocked by any orb or shadow orb.
    - Cannot move onto a non-empty goal socket unless the move is a valid end-state capture/win condition.
    - Only the current player may move their own orbs on their turn.
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
      - Verify target is within allowed move pattern (adjacent or straight-line slide).
      - Check each cell along the path is empty or capturable according to rules.
      - Ensure destination is empty or permitted by end-state rules (e.g., goal socket under conditions).
      - Return is_valid and a preview of resulting board state if valid.
**Movement:**
  **Allowed:** slide|step
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** any for slide; 1 for step
**Capture:**
  **Type:** shadow_shift
  **Directions:**
    - horizontal
    - vertical
    - diagonal
    - all
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** AI shadow orbs alter available spaces after its turn, influencing future moves
  **Capture Algorithm:**
    **Name:** apply_shadow_shift
    **Inputs:**
      - row
      - col
      - board
      - parameters
    **Outputs:**
      **Affected Count:** int
    **Parameters:**
      **Directions:** array
      **Shuffle Seed:** string
      **Radius:** int
    **Steps:**
      - Compute shadow shift using deterministic seed
      - Apply shadow positions to the board
      - Update available spaces and affected cells
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** Player with more collected/secured orbs or achieved socket goals wins; otherwise draw

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
  **Condition:** current_player == 1 AND game.status == playing AND there exists at least one valid move for current player
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
**Text:** All gameplay logic MUST be deterministic, replayable, and free of deadlocks under fixed seeds.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic-based move selection.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate-depth search (e.g., minimax with pruning).
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search strategy (e.g., minimax with deeper lookahead or MCTS).
**Type:** core


## End Conditions

**Win:**
  **Condition:** human achieves all goal sockets or accumulates sufficient points
  **Check Logic:** Evaluate after each move; if objectives are met, declare human win
  **Priority:** immediate
**Lose:**
  **Condition:** AI achieves its win condition or human is unable to move on AI-only turns
  **Check Logic:** Evaluate after each move; if AI's win conditions met or human has no legal moves for a full turn, declare AI win
  **Priority:** immediate
**Draw:**
  **Condition:** no legal moves exist for both players in a full turn or repetitive stalemate
  **Check Logic:** If no legal moves for current player and no progress over a full cycle, declare draw
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
  - HUD: score/points, remaining goals, move count
**Interactions:**
  - Click/tap to make move
  - Hover for preview/hints
  - Click/press restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser/draw modal
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
**Then:** Captured pieces/spaces update exactly as defined


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