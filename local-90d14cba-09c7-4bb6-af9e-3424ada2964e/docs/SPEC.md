# Mine Cart Grid

## Meta

**Game Name:** Mine Cart Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Navigate mine cart tokens on a 6x6 grid toward exit rails by selecting a cart then moving it to a neighboring empty cell or by dragging along a straight line; AI occasionally shelves a rock blocker to complicate paths.
**Session Minutes:**
  - 20
  - 40

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
    **Grid:** 6x6 matrix with cells containing {occupantId|null, blocked|rock} entries; initial layout places a subset of mine carts for Player 1 and AI tokens along starting zones; rocks may appear as blockers
    **Rows:** 6
    **Cols:** 6
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
  **Initial Placement:** Place mine cart tokens on designated starting rows for each player; exit rails are at opposing edges; rocks blocker tokens may be introduced by AI shelves during play. All placements are deterministic and replayable given seed.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - target cell is empty
    - move conforms to allowed movement: single-step to adjacent cell OR drag along a straight line with a continuous, unblocked path
    - drag path is straight (no turns) and terminates on an empty cell
    - cannot move into a blocked cell unless the blocker is removed by game rules
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
      - Ensure target is within allowed range (1 step or straight-line drag)
      - Check path between source and target is clear of blockers
      - Verify target is empty
      - Confirm movement respects turn order
**Movement:**
  **Allowed:** step|slide
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** step:1; slide:any
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture mechanics in core play; rocks blockers persist until moved or removed by rules
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**

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
  **Method:** points
  **Winner Determination:** The human player wins if any of their carts reach an designated exit rail; the AI wins if its cart(s) reach an exit rail; if both reach exit rails in the same turn, it's a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 20
**Actions:**
  **Name:** player_move
  **Parameters:**
    - sourceRow
    - sourceCol
    - targetRow
    - targetCol
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
**Text:** Clear, testable rule with input/output (MUST/SHOULD/CAN).
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple evaluation that prioritizes nearest exit path with minimal branching.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST consider at least two candidate moves and evaluate paths toward exit rails.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST perform deeper search with pruning and rock blocker shelf considerations.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_reaches_exit
  **Check Logic:** Any human cart on exit rail cell
  **Priority:** immediate
**Lose:**
  **Condition:** ai_reaches_exit
  **Check Logic:** Any AI cart on exit rail cell
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_sides OR both_reach_exit_simultaneously
  **Check Logic:** End-turn evaluation shows no winner
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
  - Display winner/loser modal
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


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates deterministically and logs the action


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI selects a valid move within 2 seconds and updates state deterministically


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