# rock paper scrissors

## Meta

**Game Name:** rock paper scrissors
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Rock Paper Scissors: each round both players secretly select rock, paper, or scissors and reveal simultaneously; rock beats scissors, scissors beats paper, and paper beats rock; ties occur on equal choices. The engine resolves each round deterministically and supports best-of-N or timed formats.
**Session Minutes:**
  - 5
  - 15
**Rng Seed:** seed-12345

## State

**Board:**
  **Type:** abstract
  **Topology:** none
  **Dimensions:**
    - 0
    - 0
  **Neighbors:** 
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
    **Difficulty:** medium
    **Depth:** 0
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** rule_based
    **Difficulty:** medium
    **Depth:** 0
    **Response Delay Ms:** 500
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** N/A for abstract rock-paper-scissors; no spatial board
    **Rows:** 0
    **Cols:** 0
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
**Scores:**
  **Human:** 0
  **Ai:** 0
**Current Round:** 1
**Players Order:**
  - 1
  - 2
**Player Roles:**
  **1:** human
  **2:** ai

## Mechanics

**Setup:**
  **Initial Placement:** No placement required. Each round, both players choose Rock, Paper, or Scissors.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** False
  **Validity Checks:**
    - choice in {rock, paper, scissors}
    - inputs normalized to lowercase before validation
  **Validation Algorithm:**
    **Name:** choice_validation
    **Inputs:**
      - round
      - current_player
      - board
    **Outputs:**
      **Is Valid:** True
      **Preview:** None
    **Parameters:**
      **Directions:**
        - (None)
      **Min Chain Length:** 0
      **Require Bounded:** False
      **Gravity:** False
      **Case Insensitive:** True
    **Steps:**
      - Normalize input to lowercase.
      - Verify the selected option is one of rock, paper, or scissors.
      - Ensure the round is active and the game expects a move from current_player.
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
  **Result:** Round outcome determined by standard RPS rules; scores updated; potential end-of-game evaluation.
  **Capture Algorithm:**
    **Name:** apply_round_result
    **Inputs:**
      - current_round
      - current_player
      - board
      - choices
    **Outputs:**
      **Winner:** int
      **Round Result:** string
    **Parameters:**
      **Directions:**
        - (None)
      **Min Chain Length:** 0
      **Require Bounded:** False
    **Steps:**
      - Compare the two choices using RPS rules.
      - Update per-round result and scores.
      - Advance to next round or end game if threshold reached.
**Turn Flow:**
  **Switch After Move:** True
  **Pass If No Valid Move:** False
  **Extra Turn Conditions:** end game after two consecutive passes
**Scoring:**
  **Method:** points
  **Winner Determination:** First to reach the configured number of round wins (best-of-N) or based on timed rounds; ties possible.
**Turn Timers:**
  **Human Seconds:** 30
  **Ai Seconds:** 2

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - choice
  **Condition:** current_player == 1 AND game.status == playing
  **Result:** Apply mechanics.move_validation and mechanics.capture; update scores and rounds; switch to AI; check end conditions
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI selects a deterministic choice based on rule_based strategy; apply mechanics.move_validation and mechanics.capture; update scores; switch to human; check end conditions
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
**Text:** AI with difficulty 'easy' MUST use a simple, deterministic random-looking selection strategy.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate strategy with some foresight.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an advanced strategy with deeper evaluation.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human reaches rounds_to_win
  **Check Logic:** Compare human score against rounds_to_win
  **Priority:** immediate
**Lose:**
  **Condition:** ai reaches rounds_to_win
  **Check Logic:** Compare AI score against rounds_to_win
  **Priority:** immediate
**Draw:**
  **Condition:** scores are tied after final round
  **Check Logic:** Compare scores at end
  **Priority:** end_turn
**Best Of N:** 3
**Rounds To Win:** 2

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
**Interactions:**
  - Click/tap to submit Rock, Paper, or Scissors
  - Hover for hints
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


**Given:** A valid capture exists per game rules
**When:** Player performs a capturing move
**Then:** Captured pieces/cards/territory update exactly as defined


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