# Dungeon Door Shuffle

## Meta

**Game Name:** Dungeon Door Shuffle
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Players move keys to their matching doors on a 5x5 grid by selecting a key and dragging to an adjacent cell or along a valid sequence; AI may hide a decoy door every few turns to mislead the player.
**Session Minutes:**
  - 10
  - 30

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
    **Color:** #000000
    **Pieces Played:** 0
  **Initial State:**
    **Pieces Played:** 0
  **Name:** AI
  **Properties:**
    **Id:** 2
    **Type:** ai
    **Color:** #ffffff
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
    **Grid:** 2D array of tokens; 0 = empty; positive integers = keys/doors; matching key/door pairs share an id; decoy doors flagged
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

## Mechanics

**Setup:**
  **Initial Placement:** 5x5 grid seeded with an equal number of key tokens and matching door tokens placed such that each key has a unique matching door; a subset of doors may be hidden as decoys by AI every few turns. Human (Player 1) starts first.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** True
  **Validity Checks:**
    - the moved key must end on its matching door
    - path must be a continuous sequence of adjacent cells
    - no overlapping with other keys/doors except for the target door when capturing
    - drag sequences must end on a valid matching door for each key moved in the sequence
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
      - Verify selected token is a movable key belonging to current player
      - Trace the proposed path for each key moved; ensure each step is within bounds and on empty cells or allowed target
      - Ensure final targets are their respective matching doors
      - If any move invalid, reject with error code/message
**Movement:**
  **Allowed:** step|drag_sequence
  **Directions:**
    - up
    - down
    - left
    - right
  **Range:** any for drag sequences; 1 for single-step moves
**Capture:**
  **Type:** match_placement
  **Directions:**
    - up
    - down
    - left
    - right
  **Require Sandwich:** False
  **Chain Capture:** False
  **Min Chain Length:** 1
  **Result:** When a key is moved to its matching door, the key is removed and the door is considered opened; scoring increments for the player. Decoy doors may be revealed/hidden by AI as per turn rules.
  **Capture Algorithm:**
    **Name:** apply_match_and_open
    **Inputs:**
      - row
      - col
      - current_player
      - board
      - parameters
    **Outputs:**
      **Opened Pairs:** int
      **Updated Board:** object
    **Parameters:**
      **Directions:**
        - up
        - down
        - left
        - right
      **Min Chain Length:** 1
      **Require Bounded:** False
    **Steps:**
      - Identify all key-to-door match placements in the move sequence
      - For each matched pair, remove the key token from its origin
      - Mark the target door as opened and locked state updated
      - Update the player's score by 1 per matched pair
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** True
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** player with the higher score when all pairs are matched or no legal moves remain; ties possible

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - move_payload
  **Condition:** current_player == 1 AND game.status == 'playing' AND there exists at least one valid move for Player 1
  **Result:** Apply mechanics.move_validation and mechanics.capture; update state; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == 'playing'
  **Result:** AI calculates move using algorithm; apply mechanics.move_validation and mechanics.capture; update state; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != 'playing'
  **Result:** Reset all state to initial values

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output MUST define concrete preconditions, targets, and outcomes.
**Type:** core


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple heuristic: choose the first valid move found.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate depth search with pruning to choose a scoring move.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced look-ahead strategy (e.g., minimax with multi-ply evaluation) and consider decoy door timing.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_pieces_matched == total_pairs
  **Check Logic:** All key-door pairs are successfully matched by the human side
  **Priority:** immediate
**Lose:**
  **Condition:** ai_pieces_matched == total_pairs
  **Check Logic:** All key-door pairs are successfully matched by the AI side
  **Priority:** immediate
**Draw:**
  **Condition:** no_legal_moves_for_both_players
  **Check Logic:** End-turn condition where both players have zero legal moves
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to select and move a key
  - Drag to create a move sequence
  - Hover for move previews
  - Click restart button
**Feedback:**
  - Highlight valid moves
  - Animate AI move
  - Show AI thinking state
  - Display winner/loser/draw messages
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
**Then:** Captured pieces updated exactly as defined (key removed, door opened, score incremented)


**Given:** AI's turn, difficulty='easy'
**When:** AI calculates move
**Then:** AI uses simple algorithm, makes valid move within 2s


**Given:** AI can win in 1 move
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST execute a winning move if available


**Given:** Human can win next turn
**When:** AI's turn, difficulty='medium' or 'hard'
**Then:** AI MUST avoid or block human's impending winning move when possible


## Config

**Default Difficulty:** medium
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 500
**Show Ai Thinking:** True