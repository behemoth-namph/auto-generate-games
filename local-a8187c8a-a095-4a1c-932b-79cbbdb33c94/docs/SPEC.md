# Sliding Block Labyrinth

## Meta

**Game Name:** Sliding Block Labyrinth
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A classic sliding block puzzle on a grid. Players alternate sliding blocks one cell at a time to create a path for a shared Goal token from the start position to a designated exit.
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
    **Ownerid:** 1
    **Player Number:** 1
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Ownerid:** 2
    **Player Number:** 2
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array representation with occupantId|null; each cell tracks which piece occupies it
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
  **Name:** Goal
  **Properties:**
    **Position:** array [row,col]
    **Exit:** array [row,col]
  **Initial State:**
    **Position:**
      - 0
      - 0
    **Exit:**
      - 4
      - 4
  **Name:** Blocks
  **Properties:**
    **Blocks:** array of block objects
  **Initial State:**
    **Blocks:**
      **Blockid:** 101
      **Owner:** 1
      **Position:**
        - 0
        - 1
      **Blockid:** 102
      **Owner:** 1
      **Position:**
        - 1
        - 0
      **Blockid:** 103
      **Owner:** 1
      **Position:**
        - 2
        - 0
      **Blockid:** 201
      **Owner:** 2
      **Position:**
        - 3
        - 4
      **Blockid:** 202
      **Owner:** 2
      **Position:**
        - 3
        - 3
      **Blockid:** 203
      **Owner:** 2
      **Position:**
        - 4
        - 2
**Holdings:**
  **Uses Holdings:** False
  **Note:** This game does not use player holdings.

## Mechanics

**Setup:**
  **Initial Placement:** Initial grid setup with movable blocks for both players and a Goal at the start; the exit is a designated cell on the grid.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - block ownership matches current_player
    - target cell is adjacent to the selected block
    - target cell is empty
    - move is a one-cell slide
  **Validation Algorithm:**
    **Name:** grid_move_check
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - blocks
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
      - Verify selected block belongs to current_player
      - Compute adjacent target cells in directions
      - Ensure target is within bounds and empty
      - Confirm move is a single-cell slide
      - Return updated board state as preview if valid
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
  **Result:** No captures occur; blocks simply reposition
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**

    **Parameters:**

    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after one player cannot move? If both players have no legal moves, trigger draw
**Scoring:**
  **Method:** points
  **Winner Determination:** If a player's block moves the goal to the exit, that player wins; otherwise, standard end conditions apply.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Ai Move Timeout Ms:** 2000
**Actions:**
  **Name:** player_move
  **Parameters:**
    - blockId
    - direction
  **Condition:** current_player == 1 AND game.status == playing
  **Result:** Apply mechanics.move_validation; apply movement; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and movement; update state; switch to human; check end conditions
  **Timeout Ms:** 2000
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic or random valid move.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate search strategy.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search strategy.
**Type:** core


## End Conditions

**Win:**
  **Condition:** Goal at exit
  **Check Logic:** The goal token position equals the exit coordinates
  **Priority:** immediate
**Lose:**
  **Condition:** Goal not at exit and no legal moves for current player
  **Check Logic:** On turn start, if current player has zero legal moves and goal not at exit
  **Priority:** immediate
**Draw:**
  **Condition:** No legal moves for both players in successive turns
  **Check Logic:** If a full cycle passes with no moves
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to move a block in a valid direction
  - Hover for move previews
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
    **Player 2:** #ff6b6b
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


**Given:** A valid slide exists per game rules
**When:** Player slides a block onto an empty adjacent cell
**Then:** Block position updates accordingly; board reflects new occupancy; last_move updated


**Given:** The goal token is moved to the exit
**When:** Player moves the block carrying the Goal token into the exit cell
**Then:** Win condition is satisfied and game ends with human as winner


**Given:** Invalid move attempt
**When:** Player attempts to move a block not owned by the current player or moves into a non-adjacent/blocked cell
**Then:**
  **Errorcode:** ERR_INVALID_MOVE
  **Message:** Invalid move: ownership, adjacency, or occupancy rule violated


**Given:** Blocking occurs with no legal moves for current player
**When:** Turn starts and current player has no valid slides
**Then:** Draw may occur if both players have no moves in succession


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic or random valid move and responds within 2 seconds


**Given:** AI can win in one move
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
**Rng:**
  **Algo:** xoshiro256**
  **Seed:** 123456789
  **Seed Source:** server