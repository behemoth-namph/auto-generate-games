# Dragonfly Grid

## Meta

**Game Name:** Dragonfly Grid
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Drag dragonflies on a 6x6 grid toward nectar cells. Tap to move to an adjacent empty cell, or drag to glide along a path. The AI places pond reeds to block routes, creating dynamic obstacles.
**Session Minutes:**
  - 5
  - 20

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
    **Id:** int (1 for human, 2 for AI)
    **Type:** string (human|ai)
    **Color:** string
    **Pieces Played:** int
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Algorithm:** string (random|minimax|mcts|rule_based)
    **Difficulty:** string (easy|medium|hard)
    **Depth:** int (minimax depth, if applicable)
    **Response Delay Ms:** int (for better UX)
    **Is Thinking:** boolean
  **Initial State:**
    **Algorithm:** minimax
    **Difficulty:** medium
    **Depth:** 4
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** 2D array of cells; each cell may hold a dragonfly id or null; nectar flags and reeds stored per cell
    **Rows:** int
    **Cols:** int
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** int (1 or 2)
    **Status:** string (playing|human_wins|ai_wins|draw)
    **Move Count:** int
    **Last Move:** object
    **Nectar Target:** int
    **Points:**
      **Human:** int
      **Ai:** int
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Nectar Target:** 5
    **Points:**
      **Human:** 0
      **Ai:** 0
    **Last Move:** None

## Mechanics

**Setup:**
  **Initial Placement:** Dragonflies start on predefined nectar-adjacent starting cells. Nectar cells are scattered on the grid; reeds are placed by AI over time to block routes.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - destination cell is within bounds
    - destination cell is empty
    - if path length > 1, the path must be continuous through empty cells
    - path must not cross any pond reeds
    - for glide moves, path ends on a nectar cell or an empty cell with valid continuation
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
      **Require Bounded:** True
      **Gravity:** False
    **Steps:**
      - Verify destination within bounds and empty
      - If move length == 1, ensure adjacency
      - If glide path, ensure all intermediate cells are empty and not blocked by reeds
      - Ensure end cell is allowable (empty or nectar as per move type)
      - Return is_valid and a preview of affected cells
**Movement:**
  **Allowed:** step|slide
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** step:1; slide:any (until blocked or until a nectar/end condition is reached)
**Capture:**
  **Type:** none
  **Directions:**
    - (None)
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 0
  **Result:** No capture occurs; reeds placed by AI persist as blocking obstacles
  **Capture Algorithm:**
    **Name:** n/a
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
  **Method:** points
  **Winner Determination:** First to reach nectar_target points wins; if both reach target, player with fewer moves wins; draw if no moves and no points have changed for a full round
**Blocker Introduction:**
  **Description:** AI may place pond reeds to block routes between dragonflies and nectar cells
  **Trigger Conditions:**
    - AI turn end
    - blocked_path_detected
  **Effects:**
    - Add reed to cell; update path availability; possible blocking of nectar access

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - dragonfly_id
    - destination_row
    - destination_col
  **Condition:** current_player == 1 AND game.status == playing AND move is valid per mechanics.move_validation
  **Result:** Apply mechanics.move_validation and mechanics.capture; update game state; switch to AI; check end conditions
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
**Text:** All inputs and outcomes MUST be deterministic and replayable given fixed seeds.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple distance-based heuristic to move toward nectar and avoid reeds.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a pathfinding heuristic (e.g., A* with reeds treated as obstacles) to select a near-optimal move.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST perform lookahead using minimax or MCTS with a bounded depth to evaluate nectar access and reed placement.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_points >= nectar_target
  **Check Logic:** Human dragonflies have accumulated nectar_target points
  **Priority:** immediate
  **Condition:** ai_points >= nectar_target
  **Check Logic:** AI dragonflies have accumulated nectar_target points
  **Priority:** immediate
**Lose:**
  **Condition:** Neither player reaches nectar_target within max_rounds and all dragonflies are unable to move
  **Check Logic:** End-of-round stagnation check
  **Priority:** end_turn
**Draw:**
  **Condition:** max_rounds_reached AND neither player reached nectar_target
  **Check Logic:** Round cap reached without winner
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
  - Drag to glide path for nectar-baiting moves
  - Hover for preview/hints
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/ loser messages
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


**Given:** A valid path move exists per game rules
**When:** Player performs a tapping move to an adjacent empty cell
**Then:** Dragonfly moves to the destination; board updates; reeds unaffected


**Given:** A valid glide move exists per game rules
**When:** Player drags to glide along a path to a nectar or valid end cell
**Then:** Dragonfly glides along the path; intermediate cells updated; end state valid


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple heuristic and moves within 2 seconds


**Given:** AI can place a reed blocking a route
**When:** AI ends its turn
**Then:** Reed is placed and affects subsequent path calculations


**Given:** Human can win by reaching nectar_target
**When:** Human turn achieves nectar_target
**Then:** Human wins and game ends


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True