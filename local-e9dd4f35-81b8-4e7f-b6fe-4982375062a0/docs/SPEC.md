# Battleship Studio

## Meta

**Game Name:** Battleship Studio
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Place ships on a private 10x10 ownShipGrid and then take turns firing at the opponent's public firingGrid until all ships are sunk. The AI's moves are deterministic, driven by a fixed rng_seed to allow exact replay and testing.
**Session Minutes:**
  - 15
  - 60

## State

**Board:**
  **Type:** grid
  **Topology:** square
  **Dimensions:**
    - 10
    - 10
  **Neighbors:** up, down, left, right
  **Boards:**
    **Player1:**
      **Ownshipgrid:**
        **Rows:** 10
        **Cols:** 10
        **Cells:**
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        **Visibility:** private
      **Firinggrid:**
        **Rows:** 10
        **Cols:** 10
        **Cells:**
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        **Visibility:** public
    **Player2:**
      **Ownshipgrid:**
        **Rows:** 10
        **Cols:** 10
        **Cells:**
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        **Visibility:** private
      **Firinggrid:**
        **Rows:** 10
        **Cols:** 10
        **Cells:**
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
          - [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        **Visibility:** public
  **Visibility Rules:** ownShipGrid private to respective owner; firingGrid visible to both players; opponent ships hidden on ownShipGrid
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** 1
    **Type:** human
    **Color:** #000000
    **Ships Placed:** 0
    **Ships Sunk:** 0
    **Shots Taken:** 0
  **Initial State:**
    **Ships Placed:** 0
    **Ships Sunk:** 0
    **Shots Taken:** 0
  **Name:** AI
  **Properties:**
    **Id:** 2
    **Type:** ai
    **Color:** #ffffff
    **Ships Placed:** 0
    **Ships Sunk:** 0
    **Shots Taken:** 0
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Rng Seed:** 12345
    **Is Thinking:** False
  **Initial State:**
    **Ships Placed:** 0
    **Ships Sunk:** 0
    **Shots Taken:** 0
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Rng Seed:** 12345
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** per-player grids are defined in the boards section above; this field describes the relationship to per-player grids for placement and firing
    **Rows:** 10
    **Cols:** 10
    **Id:** board_1
    **Coord:**
      **Row:** 0
      **Col:** 0
    **Tags:**
      - battlefield
      - grid
    **Occupantid:** None
    **Visibility:** private
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** 1
    **Status:** placing
    **Move Count:** 0
    **Last Move:**

  **Initial State:**
    **Current Player:** 1
    **Status:** placing
    **Move Count:** 0
    **Last Move:**


## Mechanics

**Setup:**
  **Initial Placement:** Human places ships on their own grid. Standard fleet: Carrier(5), Battleship(4), Cruiser(3), Submarine(3), Destroyer(2). Ships must fit within 10x10 and cannot overlap. Drag to set length and orientation or click cells to start and end positions. After all ships are placed, switch to firing mode.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - ships_do_not_overlap
    - ships_within_bounds
    - each_ship_placed_exactly_once
    - shots_are_on_opponent_grid_and_not_previously_taken
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
        - horizontal
        - vertical
      **Min Chain Length:** 2
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Locate target cell in current phase (placement or firing).
      - If placement, verify ship does not overlap any existing ship and fits within bounds.
      - If firing, verify target cell has not been shot previously on the opponent's firing grid.
      - Return is_valid and a preview of affected cells if placement.
**Movement:**
  **Allowed:** none
  **Directions:**
    - (None)
  **Range:** 0
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** Marks a fired cell as hit or miss and updates ship state; ships sink when all cells are hit.
  **Capture Algorithm:**
    **Name:** apply_hit_result
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Affected Count:** int
      **State:** object
    **Parameters:**
      **Directions:**
        - (None)
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - Determine target cell from move.
      - Check opponent ship presence at that cell.
      - If hit, mark cell, update ship hit-segments; if ship fully hit, mark sunk.
      - If miss, mark miss on firing grid.
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after both players have declared no valid moves or one side is completely sunk
**Scoring:**
  **Method:** sinking_all_opponent_ships
  **Winner Determination:** The player who sinks all of the opponent's ships wins. If both sides are sunk simultaneously due to a quirk, declare a draw.

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - row
    - col
  **Condition:** current_player == 1 AND game.status != 'game_over'
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status != 'game_over'
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status == 'game_over'
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST/CAN/SHOULD notation.
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use seeded RNG with rng_seed; moves are deterministic given the seed.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate heuristic (targeted search after a hit).
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced strategy (probability density / deeper search within seed).
**Type:** core


## End Conditions

**Win:**
  **Condition:** opponent_ships_all_sunk
  **Check Logic:** All ship segments of the opponent are marked sunk
  **Priority:** immediate
**Lose:**
  **Condition:** current_player_ships_all_sunk
  **Check Logic:** All ship segments of the current player are marked sunk
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_sides
  **Check Logic:** Both players have no valid firing targets and no ships sunk this round
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
**When:** Player performs a firing move
**Then:** Hit/miss and possible sink reflect in state exactly


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses seeded RNG with rng_seed and acts within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST not block but allow human to win on subsequent turn if deterministic outcome dictates


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True
**Rng Seed:** 12345
**Difficulty Map:**
  **Easy:** seeded_random
  **Medium:** heuristic_seeded
  **Hard:** deep_search_seeded