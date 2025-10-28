# rock paper scrissors

## Meta

**Game Name:** rock paper scrissors
**Game Type:** board
**Player Mode:** player_vs_ai
**Players:**
  **Human:** 1
  **Ai:** 1
**Core Mechanic:** Rock, Paper, Scissors with simultaneous reveal implemented as a turn-based round system; best-of-three rounds to determine the winner. All gameplay is deterministic and replayable given fixed inputs/seeds.
**Session Minutes:**
  - 1
  - 5

## State

**Board:**
  **Type:** free
  **Topology:** none
  **Dimensions:**
    - 0
    - 0
  **Neighbors:** none
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
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 300
    **Is Thinking:** False
  **Initial State:**
    **Algorithm:** rule_based
    **Difficulty:** easy
    **Depth:** 1
    **Response Delay Ms:** 300
    **Is Thinking:** False
  **Name:** Board
  **Properties:**
    **Grid:** abstract_move_space
    **Rows:** 0
    **Cols:** 0
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Last Move:** None
  **Initial State:**
    **Current Player:** 1
    **Status:** playing
    **Move Count:** 0
    **Human Wins:** 0
    **Ai Wins:** 0
    **Round:** 0

## Mechanics

**Setup:**
  **Initial Placement:** Each round consists of both players secretly selecting rock, paper, or scissors; no spatial placement required.
  **Starting Player Rule:** human
**Move Validation:**
  **Must Place On Empty:** False
  **Validity Checks:**
    - move must be one of: rock, paper, scissors
  **Validation Algorithm:**
    **Name:** choices_check
    **Inputs:**
      - current_move
      - state
    **Outputs:**
      **Is Valid:** boolean
      **Preview:** object|null
    **Parameters:**
      **Allowedmoves:**
        - rock
        - paper
        - scissors
    **Steps:**
      - Check if current_move is in allowedMoves
      - If valid, set is_valid to true and set preview to the chosen move
      - If invalid, set is_valid to false and provide an error response
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
  **Result:** no captures in RPS
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
  **Extra Turn Conditions:** end game after two round wins by either side
**Scoring:**
  **Method:** points
  **Winner Determination:** After each round, increment human_wins or ai_wins based on the round winner; first to 2 wins the match; if 3 rounds played with no 2-win result, declare a draw

## Turns

**Order:** Human (Player 1) → AI (Player 2) → repeat
**Max Time Seconds:** 30
**Actions:**
  **Name:** player_move
  **Parameters:**
    - move
  **Condition:** current_player == 1 AND game.status == playing
  **Result:** Record human's move for the current round; wait for AI move; after AI move, determine round winner; update scores; increment round; switch to AI or end game if win condition reached
  **Name:** ai_move
  **Parameters:**
    - (None)
  **Condition:** current_player == 2 AND game.status == playing
  **Result:** AI selects move using rule_based algorithm; apply move; compute round result against human_move; update scores; increment round; switch to human; check end conditions
  **Name:** restart
  **Parameters:**
    - (None)
  **Condition:** game.status != playing
  **Result:** Reset all state to initial values; prepare a fresh best-of-three match

## Rules


**Id:** R1
**Text:** Clear, testable rule with input/output (MUST/SHOULD/CAN)
**Type:** core|validation|optional


**Id:** R_AI_1
**Text:** AI MUST make a move within 2 seconds of its turn starting.
**Type:** core


**Id:** R_AI_2
**Text:** AI with difficulty 'easy' MUST use a simple, deterministic choice among rock/paper/scissors.
**Type:** core


**Id:** R_AI_3
**Text:** AI with difficulty 'medium' MUST use a moderate rule-based strategy.
**Type:** core


**Id:** R_AI_4
**Text:** AI with difficulty 'hard' MUST use an enhanced strategy with predictive tendencies.
**Type:** core


## End Conditions

**Win:**
  **Condition:** human_wins >= 2
  **Check Logic:** Human has reached two round wins (best-of-three).
  **Priority:** immediate
**Lose:**
  **Condition:** ai_wins >= 2
  **Check Logic:** AI has reached two round wins (best-of-three).
  **Priority:** immediate
**Draw:**
  **Condition:** round >= 3 AND human_wins < 2 AND ai_wins < 2
  **Check Logic:** Three rounds completed with neither side reaching two wins.
  **Priority:** end_turn

## Ui

**Display Elements:**
  - Board/game area
  - Current player indicator (You vs AI)
  - AI thinking indicator
  - Game status message
  - Restart button
  - Scoreboard
**Interactions:**
  - Click/tap to submit move
  - Keyboard shortcuts for rock/paper/scissors
  - Hover for focus/preview
  - Click restart button
**Feedback:**
  - Highlight chosen move
  - Animate AI move
  - Show AI thinking state
  - Display winner message
  - Update score immediately after each round
**Board Style:**
  **Cell Size:** 64
  **Grid Line Color:** #222222
  **Disc Colors:**
    **Player 1:** #1e90ff
    **Player 2:** #ff4500
    **Outline:** #888888
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
**Then:** Human move is recorded; AI move computed; round result computed and displayed; scores update; next round begins


**Given:** A valid human move
**When:** AI calculates move at easy/ deterministically
**Then:** AI uses the rule-based algorithm and responds within 2s


**Given:** Two round wins by either side
**When:** End conditions are checked
**Then:** Game ends with the appropriate winner modal


**Given:** Three rounds completed with no winner
**When:** End conditions checked
**Then:** Draw state is declared and displayed


## Config

**Default Difficulty:** easy
**Allow Difficulty Change:** True
**Ai Response Delay Ms:** 300
**Show Ai Thinking:** True
**Rng Related:**
  **Prng Algo:** xoshiro256**
  **Seed Source:** server_seed