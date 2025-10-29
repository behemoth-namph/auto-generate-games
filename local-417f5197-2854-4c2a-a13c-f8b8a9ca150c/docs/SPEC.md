# Mini Chess Grid

## Meta

**Game Name:** Mini Chess Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** A deterministic, turn-based 6x6 grid-chess variant. The human player selects a piece and moves to a target square; captures remove opponent pieces; turns alternate; drag-to-move is supported for tactile input.
**Session Minutes:**
  - 5
  - 60

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
    - up-left
    - up-right
    - down-left
    - down-right
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
    **Grid:** 6x6 2D array; each cell is an object with {row, col, occupantId|null, visible, tags}
    **Rows:** 6
    **Cols:** 6
  **Initial State:**
    **Occupancy Map:**
      - [{'row': 0, 'col': 0, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 0, 'col': 1, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 0, 'col': 2, 'occupantId': 'K1', 'visible': True, 'tags': ['king']}, {'row': 0, 'col': 3, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 0, 'col': 4, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 0, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
      - [{'row': 1, 'col': 0, 'occupantId': 'P1-1', 'visible': True, 'tags': ['pawn']}, {'row': 1, 'col': 1, 'occupantId': 'P1-2', 'visible': True, 'tags': ['pawn']}, {'row': 1, 'col': 2, 'occupantId': 'P1-3', 'visible': True, 'tags': ['pawn']}, {'row': 1, 'col': 3, 'occupantId': 'P1-4', 'visible': True, 'tags': ['pawn']}, {'row': 1, 'col': 4, 'occupantId': 'P1-5', 'visible': True, 'tags': ['pawn']}, {'row': 1, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
      - [{'row': 2, 'col': 0, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 2, 'col': 1, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 2, 'col': 2, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 2, 'col': 3, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 2, 'col': 4, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 2, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
      - [{'row': 3, 'col': 0, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 3, 'col': 1, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 3, 'col': 2, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 3, 'col': 3, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 3, 'col': 4, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 3, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
      - [{'row': 4, 'col': 0, 'occupantId': 'PAI-1', 'visible': True, 'tags': ['pawn']}, {'row': 4, 'col': 1, 'occupantId': 'PAI-2', 'visible': True, 'tags': ['pawn']}, {'row': 4, 'col': 2, 'occupantId': 'PAI-3', 'visible': True, 'tags': ['pawn']}, {'row': 4, 'col': 3, 'occupantId': 'PAI-4', 'visible': True, 'tags': ['pawn']}, {'row': 4, 'col': 4, 'occupantId': 'PAI-5', 'visible': True, 'tags': ['pawn']}, {'row': 4, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
      - [{'row': 5, 'col': 0, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 5, 'col': 1, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 5, 'col': 2, 'occupantId': 'K2', 'visible': True, 'tags': ['king']}, {'row': 5, 'col': 3, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 5, 'col': 4, 'occupantId': None, 'visible': True, 'tags': []}, {'row': 5, 'col': 5, 'occupantId': None, 'visible': True, 'tags': []}]
    **Rows:** 6
    **Cols:** 6
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
  **Initial Placement:** Canonical starting state derived from occupancy_map; starting_pieces data removed. Starting arrangement mirrors both sides with pawns on row 1 and king-like pieces on row 0/5; human moves first.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** False
  **Validity Checks:**
    - the selected piece belongs to the current player
    - target square is empty OR occupied by opponent
    - target move matches the piece's move rules (king-like and pawn-like behavior in this simplified variant)
    - target stays within board bounds
    - move does not leave own king in check (simplified rule set)
  **Pawn Orientation:**
    **Human:**
      **Forward:** -1
      **Captures:**
        **Dr:** -1
        **Dc:** -1
        **Dr:** -1
        **Dc:** 1
    **Ai:**
      **Forward:** 1
      **Captures:**
        **Dr:** 1
        **Dc:** -1
        **Dr:** 1
        **Dc:** 1
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
        - up-left
        - up-right
        - down-left
        - down-right
      **Min Chain Length:** 1
      **Require Bounded:** True
      **Gravity:** False
      **Is Capture:** True
    **Steps:**
      - Ensure a piece exists at the origin and belongs to current_player
      - Ensure target is within allowed movement pattern for the piece
      - Ensure target is empty OR occupied by opponent
      - Validate that path (if sliding) is unobstructed
      - Return validity and a preview of resulting board state if valid
**Movement:**
  **Allowed:** step|slide
  **Directions:**
    - up
    - down
    - left
    - right
    - up-left
    - up-right
    - down-left
    - down-right
  **Range:** 1
**Capture:**
  **Type:** piece_removal
  **Directions:**
    - adjacent diagonals or orthogonal depending on piece
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** Opponent piece at target square is removed; moving piece occupies target
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
        - adjacent diagonals
        - orthogonals
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - Identify target cell occupancy by opponent
      - Remove occupant
      - Move active piece to target cell
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** count_pieces
  **Winner Determination:** Player with more pieces at end of game or when opponent has zero pieces wins

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Ai Turn Time Limit Ms:** 2000
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceRow
    - sourceCol
    - targetRow
    - targetCol
  **Condition:** current_player == 1 AND game.status == playing AND [move validity conditions]
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
**Text:** Clear, testable rule with input/output (MUST/SHOULD/CAN)
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic: pick the first legal move found.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate search (e.g., depth-limited minimax).
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced search (e.g., deeper minimax with pruning).
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_wins
  **Check Logic:** human captures all AI pieces or AI has zero pieces; verify at end of turn
  **Priority:** immediate
**Lose:**
  **Condition:** ai_wins
  **Check Logic:** AI captures all human pieces or human has zero pieces; verify at end of turn
  **Priority:** immediate
**Draw:**
  **Condition:** no legal moves for both players or repetition detected or move limit reached
  **Check Logic:** assess at end of turn
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
  - Drag to move (drag-to-move enabled)
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
**Then:** Captured pieces update exactly as defined


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's winning move


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Ai Turn Time Limit Ms:** 2000