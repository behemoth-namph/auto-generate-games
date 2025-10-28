# Moonlit Mines

## Meta

**Game Name:** Moonlit Mines
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Turn-based 6x6 grid where the human scout moves to adjacent safe cells while the AI secretly places mine tokens to impede progress; the human objective is to reach the far exit while avoiding mines. AI increases tension by adding one mine token every few moves.
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
    - N
    - S
    - E
    - W
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
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 2
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 2
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array of cells with occupant and mine flags; pre-seeded mine layout is deterministic per seed
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
  **Initial Placement:** Human scout starts at (0,0). The board is pre-seeded with a deterministic distribution of mine tokens and safe cells using the game seed. The AI starts with 0 additional mines and will place mines as the game progresses.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination must be adjacent to the scout's current position
    - destination cell must be safe (not currently a mine)
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
        - N
        - S
        - E
        - W
      **Min Chain Length:** 1
      **Require Bounded:** False
      **Gravity:** False
    **Steps:**
      - Identify current scout position for current_player.
      - Validate destination is neighboring in one of the allowed directions.
      - Check destination is within bounds and not occupied by a mine.
      - Return is_valid and a preview of the new state if valid.
**Movement:**
  **Allowed:** step|drag
  **Directions:**
    - N
    - S
    - E
    - W
  **Range:** step = 1; drag = up to 2 cells
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture occurs via movement; mines are controlled by AI
  **Capture Algorithm:**
    **Name:** none
    **Inputs:**
      - (None)
    **Outputs:**
      **Affected Count:** 0
    **Parameters:**

    **Steps:**
      - (None)
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** The player with more points at game end wins; points awarded for safe progress toward exit and successful moves while mines impose penalties or risk.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
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
**Text:** All gameplay MUST be deterministic and replayable under fixed seeds.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic to place mines and select safe near-term moves.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate heuristic to balance mine placement and path disruption.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced heuristic to maximize tension while preserving solvability.
**Type:** core


**Id:** R_AI_MINE_1
**Text:** AI MUST add exactly one new mine token after every three human moves (except when end conditions occur).
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_reaches_exit
  **Check Logic:** Player 1's scout position equals exit cell (5,5) and the cell is not mined
  **Priority:** immediate
**Lose:**
  **Condition:** scout_destroyed_by_mine
  **Check Logic:** Scout token is removed or destroyed due to mine explosion or capture
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_available
  **Check Logic:** End turn with no valid moves for both sides
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
**Aria:**
  **Cell Label Template:** Cell {row},{col} - {content}

## Technical

**Platform:** HTML5 Canvas
**Performance:**
  **Target Fps:** 60
  **Max Load Time Ms:** 1500
  **Max Ai Think Time Ms:** 2000
**Seed Preservation:** True
**Determinism Enforcement:** True

## Acceptance


**Given:** Initial state description
**When:** Human action performed
**Then:** Expected result with specific values


**Given:** A valid move exists per game rules
**When:** Player performs a valid move
**Then:** State updates correctly: scout position, move_count increments, last_move recorded


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic and makes a valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST not block an inevitable human win if that would violate deterministic rules; otherwise it must block if a blocking move exists


## Config

**Default Difficulty:** easy
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Rng:**
  **Algorithm:** xoshiro256**
  **Seed:** MoonlitSeed-001
**Ui:**
  **Hit Target Min Px:** 40
  **Hit Target Padding Px:** 8