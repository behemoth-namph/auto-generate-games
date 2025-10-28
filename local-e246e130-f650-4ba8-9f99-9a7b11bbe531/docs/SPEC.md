# Checkers

## Meta

**Game Name:** Checkers
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Two-player checkers on an 8x8 board where pieces move diagonally, capture by jumping opponent pieces, and promote to kings; you select a piece by clicking it, then click a valid destination to move; captures are forced when available and AI uses a simple rule-based strategy.
**Session Minutes:**
  - 15
  - 60

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 8
    - 8
  **Neighbors:** for grid: directions list; for hex: 6 directions; for graph: adjacency function
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** dark
    **Pieces Played:** 0
    **Player Number:** 1
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Player Number:** 2
  **Initial State:**
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 500
    **Is Thinking:** False
    **Time Budget Ms:** 2000
  **Name:** Board
  **Properties:**
    **Grid:** 8x8 grid with pieces placed on dark squares; internal representation as 2D array of cells with pieceId or null
    **Rows:** 8
    **Cols:** 8
  **Initial State:**
    **Grid:**
      - [None, 20001, None, 20002, None, 20003, None, 20004]
      - [20005, None, 20006, None, 20007, None, 20008, None]
      - [None, 20009, None, 20100, None, 20101, None, 20102]
      - [None, None, None, None, None, None, None, None]
      - [None, None, None, None, None, None, None, None]
      - [10001, None, 10002, None, 10003, None, 10004, None]
      - [None, 10005, None, 10006, None, 10007, None, 10008]
      - [10009, None, 10100, None, 10101, None, 10102, None]
    **Placement:** Standard starting position: AI on dark squares rows 0-2; human on dark squares rows 5-7; light squares empty.
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
  **Initial Placement:** Each side starts with 12 pieces on dark squares of the first three rows (AI on top, human on bottom). Pieces move diagonally forward; kings are promoted on reaching the far rank.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - current_player owns the moving piece
    - destination is a dark square
    - destination square is empty
    - move is either a single-diagonal step (for non-king) or a capturing jump sequence
    - if any capture is available for the current player, only capture moves are valid
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
        - NW
        - NE
        - SW
        - SE
      **Min Chain Length:** 1
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Compute all legal moves for current_player from board state
      - If any capture moves exist, filter to captures only
      - Verify destination coordinates exist and are empty
      - Return is_valid and possible preview for UI hints
**Movement:**
  **Allowed:** step|jump|slide
  **Directions:**
    - NW
    - NE
    - SW
    - SE
  **Range:** step: 1 (diagonal forward for men, both directions for kings); jump: 2 (capture over adjacent opponent into empty landing)
**Capture:**
  **Type:** jump_capture
  **Directions:**
    - NW
    - NE
    - SW
    - SE
  **Require Sandwich:** True
  **Chain Capture:** True
  **Min Chain Length:** 1
  **Result:** Captured opponent piece(s) removed; if piece reaches far rank, promote to king; turn ends or continues if multi-capture path available
  **Capture Algorithm:**
    **Name:** execute_jump_chain
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Affected Count:** int
      **Updated Board:** object
    **Parameters:**
      **Directions:**
        - NW
        - NE
        - SW
        - SE
      **Min Chain Length:** 1
      **Require Bounded:** True
    **Steps:**
      - Validate initial jump capture path
      - Apply capture by removing jumped piece
      - Move capturing piece to landing square
      - If further captures are available for that piece, continue chain
      - Promote to king if landing on back rank
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after no valid moves for both sides
**Scoring:**
  **Method:** count_pieces
  **Winner Determination:** Player with at least one piece remaining when opponent cannot move; otherwise the player with more pieces when game ends

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceRow
    - sourceCol
    - destRow
    - destCol
  **Condition:** current_player == 1 AND game.status == playing AND move is valid
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI calculates move using rule_based logic; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output (MUST) describing a game action's validity and effect.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple rule-based evaluation focusing on capturing when available and advancing toward opponent.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST consider basic piece safety and king potential in its chosen capture/move sequence.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST implement more advanced heuristics (positional value, king safety) within the deterministic rules.
**Type:** core


## End Conditions

**Win:**
  **Condition:** AI_pieces == 0 OR AI_no_legal_moves
  **Check Logic:** If the opponent (AI) has zero pieces or cannot make a legal move on the human turn, human wins.
  **Priority:** immediate
**Lose:**
  **Condition:** Player_pieces == 0 OR Player_no_legal_moves
  **Check Logic:** If the human player cannot move or has zero pieces, AI wins.
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_players
  **Check Logic:** Stalemate when neither side has a legal move on their turn; no captures available and no progress possible
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
**Then:** Captured pieces/cards/territory update exactly as defined


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple rule-based approach and moves within 2 seconds


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST block human's winning move if possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True