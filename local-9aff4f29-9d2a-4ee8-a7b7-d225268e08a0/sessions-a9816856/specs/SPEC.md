# Game Specification

## Ui

**Feedback:**
  - Highlight selected cell
  - Animate digit entry (fade/slide)
  - Show invalid move feedback (red shake)
  - Victory animation (celebration glow)
  - Hint visualization (subtle glow on suggested cell)
  - Move counter update
**Game Flow:**
  **Screens:**
    **Type:** required
    **Screen:** mode_selection
    **Priority:** 1
    **Description:** First screen - user selects game mode before playing
    **Interactions:**
      - Click mode card to select mode
      - Selected mode is highlighted with selected style
      - Click Start button to begin game with selected mode
    **Skip Allowed:** False
    **Display Elements:**
      - Game title/logo
      - Mode selection cards/buttons (one per available_mode)
      - Mode icon for each mode
      - Mode name for each mode
      - Mode description for each mode
      - Player count indicator
      - Config preview (time limits, difficulty, etc)
      - Start/Play button (disabled until mode selected)
    **Type:** required
    **Screen:** gameplay
    **Priority:** 2
    **Description:** Main game screen where actual gameplay occurs
    **Display Elements:**
      - Puzzle grid area (9x9 grid with bold 3x3 boxes)
      - Current mode indicator badge
      - Change Mode button
      - Hints remaining indicator
      - Undo button
      - Redo button
      - Timer (if timed mode)
      - Live validation status indicator
      - Status banner (valid/invalid/solved)
    **Type:** required
    **Screen:** game_over
    **Priority:** 3
    **Description:** End screen showing results
    **Display Elements:**
      - Result message (win/lose/draw)
      - Final score/stats
      - Play Again button (restarts with same mode)
      - Change Mode button (returns to mode_selection)
**Interactions:**
  - Click to select a cell
  - Click a digit to begin entry
  - Type a digit (1-9) to place into selected cell
  - Click a digit again to overwrite (if allowed by mode)
  - Use Undo/Redo to navigate history
  - Click Hint to reveal a single suggested cell
  - Swipe/tap for mobile: tap-diddle (tap-tap for digit entry)
**Visual Style:**
  **Background:** #f6f7f9
  **Grid Color:** #2c2c2c
  **Piece Colors:**
    **Digits:**
      **Default:** #1e88e5
      **Pencil:** #9e9e9e
  **Highlight Color:** #ffd54f
**Mode Switcher:**
  **Position:** top-right
  **Required:** True
  **Button Text:** Change Mode
  **Description:** Button to change mode during gameplay
  **Confirmation Modal:**
    **Title:** Change Game Mode?
    **Buttons:**
      **Text:** Cancel
      **Style:** secondary
      **Action:** close_modal
      **Text:** Change Mode
      **Style:** warning
      **Action:** return_to_mode_selection_and_reset_game
    **Message:** Your current game progress will be lost. Are you sure you want to continue?
    **Required:** True
**Mode Indicator:**
  **Display:**
    **Icon:** from available_modes[].icon (optional)
    **Style:**
      **Color:** #ffffff
      **Padding:** 8px 12px
      **Font Size:** 12
      **Background:** #37474f
      **Font Weight:** bold
      **Border Radius:** 4
    **Format:** Mode: {{mode_name}}
  **Position:** top-right
  **Required:** True
  **Description:** Shows current active mode during gameplay
**Mode Selection:**
  **Layout:**
    **Gap:** 20
    **Type:** grid
    **Align:** center
    **Columns:** auto-fit
    **Responsive:** True
  **Required:** True
  **Description:** Mode selection screen shown FIRST before any gameplay
  **Interactions:**
    - Click mode card to select
    - Only one mode can be selected at a time
    - Start button becomes enabled after selection
    - Click Start to transition to gameplay
  **Skip Allowed:** False
  **Mode Card Style:**
    **Hover:**
      **Border:** 2px solid #4CAF50
      **Cursor:** pointer
      **Box Shadow:** 0 4px 8px rgba(0,0,0,0.15)
    **Width:** 320
    **Height:** 180
    **Padding:** 18
    **Selected:**
      **Scale:** 1.04
      **Border:** 3px solid #45a049
      **Background:** #c8e6c9
      **Box Shadow:** 0 4px 8px rgba(76, 175, 80, 0.3)
    **Unselected:**
      **Scale:** 1.0
      **Border:** 1px solid #ddd
      **Background:** #ffffff
      **Box Shadow:** 0 2px 4px rgba(0,0,0,0.1)
    **Border Radius:** 8
  **Display Elements:**
    - Game title
    - Mode cards/buttons (dynamically generated from available_modes array)
    - For each mode: icon, name, description, player count, config preview
    - Start button (disabled until mode selected)
    - Settings button (optional)
**Display Elements:**
  - Change Mode button (during gameplay)
  - Current mode indicator (during gameplay)
  - Mode selection screen (REQUIRED - shown first before gameplay)
  - Puzzle grid
  - Move counter
  - Timer (if timed mode)
  - Hints remaining indicator
  - Undo button
  - Hint button
  - Reset button
  - Pause button
  - Victory message
**Meta:**
  **Game Name:** string
  **Game Type:** puzzle
  **Default Mode:** classic
  **Core Mechanic:** Solve Sudoku puzzles by filling 9x9 grid so each row, column, and 3x3 box contains digits 1-9 exactly once
  **Available Modes:**
    **Mode:** classic
    **Name:** Classic Sudoku
    **Config:**
      **Move Limit:** None
      **Time Limit:** None
      **Undo Allowed:** True
      **Hints Available:** 3
    **Players:**
      **Ai:** 0
      **Human:** 1
    **Description:** Solve Sudoku at your own pace
    **Player Mode:** single_player
  **Session Minutes:**
    - 5
    - 30
  **Supports Multiple Modes:** False

## Rules


**Id:** R1
**Text:** Move MUST be validated before being applied to board.
**Type:** core


**Id:** R2
**Text:** Move counter MUST increment after each valid move.
**Type:** core


**Id:** R3
**Text:** Puzzle MUST be solvable from initial state.
**Type:** core


**Id:** R4
**Text:** Win condition MUST be checked after each move.
**Type:** core


**Id:** R5
**Text:** Undo SHOULD restore exact previous state.
**Type:** validation


**Id:** R6
**Text:** Hints SHOULD be helpful without giving complete solution.
**Type:** optional


## State

**Board:**
  **Type:** grid
  **Cells:** 2D array of cell states
  **Topology:** square
  **Dimensions:**
    - 9
    - 9
**Entities:**
  **Name:** Player
  **Properties:**
    **Id:** int
    **Hints Used:** int
    **Moves Made:** int
    **Time Elapsed Ms:** int
  **Initial State:**
    **Hints Used:** 0
    **Moves Made:** 0
    **Time Elapsed Ms:** 0
  **Name:** Board
  **Properties:**
    **Cols:** int
    **Grid:** 2D array of digits or empty placeholders
    **Rows:** int
    **Empty Cell:** object {row, col} (if applicable)
    **Goal State:** 9x9 grid representing solution
  **Initial State:**
    **Grid:**
      - (None)
  **Name:** Piece
  **Note:** In Sudoku, cells hold digits 1-9 or empties
  **Properties:**
    **Id:** int
    **Row:** int
    **Col:** int
    **Type:** string (digit|empty|note)
    **Value:** int|string
    **Is Movable:** boolean
  **Initial State:**

  **Name:** Game
  **Properties:**
    **Level:** int
    **Status:** string (playing|paused|solved|failed)
    **Move Count:** int
    **Move Limit:** int|null
    **Is Solvable:** boolean
    **Mode Config:** object (stores active mode configuration)
    **Current Mode:** string (classic)
    **Move History:** array of move objects
    **Hints Remaining:** int
    **Time Remaining Seconds:** int|null
  **Initial State:**
    **Level:** 1
    **Status:** playing
    **Move Count:** 0
    **Current Mode:** classic
    **Hints Remaining:** 3

## Turns

**Type:** turn_based_single_player
**Actions:**
  **Name:** place_digit
  **Result:** Place a digit (1-9) into the selected cell if valid; updates board, increments move_count, clears conflicting pencil notes if a direct digit is placed, and checks for completion.
  **Preconditions:**
    - game.status == playing
    - a cell is selected
    - digit in 1-9
    - target cell is empty OR mode allows overwriting
    - digit does not create row/column/box conflicts
  **Postconditions:**
    - cell value updated to digit
    - move_count incremented by 1
    - conflicting notes in same row/column/box may be removed
    - win/solved checked
  **Parameters:**
    - row
    - col
    - digit
  **Name:** pencil_note
  **Result:** Toggle a pencil note (candidate) in the selected cell; does not affect actual value.
  **Preconditions:**
    - game.status == playing
    - a cell is selected
  **Postconditions:**
    - note candidate toggled for that cell
    - no change to move_count or board final values
  **Parameters:**
    - row
    - col
    - candidate
  **Name:** undo
  **Result:** Restore previous board state (if available).
  **Preconditions:**
    - move_history.length > 0
    - undo_allowed == true
  **Postconditions:**
    - board state rolled back to previous
    - move_count decremented accordingly
  **Parameters:**
    - (None)
  **Name:** hint
  **Result:** Reveal a single valid digit for a cell (without solving the puzzle).
  **Preconditions:**
    - game.status == playing
    - hints_remaining > 0
  **Postconditions:**
    - hints_remaining decremented by 1
    - one cell receives a helpful hint (suggested digit shown as pencil note or temporary highlight)
  **Parameters:**
    - (None)
  **Name:** reset
  **Result:** Reset board to initial puzzle state, clear move history, reset hints.
  **Preconditions:**
    - always
  **Postconditions:**
    - board reset to initial grid
    - move_history cleared
    - hints_remaining reset to initial value
  **Parameters:**
    - (None)
  **Name:** pause
  **Result:** Pause the game, timer (if any) paused.
  **Preconditions:**
    - game.status == playing
  **Postconditions:**
    - game.status set to paused
  **Parameters:**
    - (None)

## Config

**Enable Undo:** True
**Enable Hints:** True
**Enable Timer:** False
**Shuffle Algorithm:** solvable_seeded
**Default Difficulty:** medium

## Mechanics

**Undo:**
  **Limit:** unlimited|N moves
  **Enabled:** True
  **Restores:** Previous board state and move count
**Hints:**
  **Cost:** 1
  **Type:** next_move|highlight_movable|show_solution_step
  **Effect:** Reveal helpful information without solving
**Setup:**
  **Randomization:** Generate a solvable Sudoku puzzle state
  **Initial Placement:** Load or generate puzzle with a unique solution
**Movement:**
  **Type:** place_digit|pencil_note
  **Rules:** Digits placed must not violate Sudoku constraints in their row, column, or 3x3 box
  **Adjacent Only:** false
**Goal Detection:**
  **Method:** state_comparison|condition_check
  **Win Condition:** Board matches target solution grid (all cells filled with 1-9 and no conflicts)
**Move Validation:**
  **Checks:**
    - Move is within bounds (row/col in 0..8)
    - Target cell accepts the action (empty or allowed overwrite for digits)
    - Digit placement does not create row/column/box conflicts
    - Pencil notes do not affect final state
  **Validation Algorithm:**
    **Name:** rule_check
    **Steps:**
      - 1. Confirm cell coordinates are valid
      - 2. If action is place_digit, verify Sudoku constraints are satisfied after placement
      - 3. If action is pencil_note, ensure candidate is in 1-9
      - 4. Return validation result
    **Inputs:**
      - row
      - col
      - action
      - board_state
    **Outputs:**
      **Reason:** string
      **Is Valid:** boolean

## End Conditions

**Win:**
  **Priority:** immediate
  **Condition:** All cells filled with valid digits and no Sudoku conflicts
  **Check Logic:** board.grid matches solution or satisfies Sudoku completion criteria
**Draw:**
  - (None)
**Lose:**
  - (None)

## Technical

**Platform:** HTML5 Canvas
**Render Engine:** PixiJS
**Description:** Rendering MUST be implemented with PixiJS on HTML5 Canvas. Use PIXI.Application, PIXI.Container, PIXI.Sprite, PIXI.Graphics, PIXI.Text, and animate with PIXI.Ticker.
**Performance:**
  **Target Fps:** 60
  **Max Load Time Ms:** 1000
**Solvability Check:** BFS|DFS|heuristic

## Acceptance


**Then:** mode_selection screen is displayed, gameplay screen is NOT visible, no mode is pre-selected
**When:** Game initialization completes
**Given:** User opens/loads game for first time


**Then:** One mode card (Classic Sudoku) is highlighted with selected styling, Start button becomes enabled
**When:** User selects the Classic Sudoku mode
**Given:** mode_selection screen is displayed with 1 mode


**Then:** Game transitions to gameplay screen, game initializes with Classic Sudoku mode config, mode_indicator shows 'Mode: Classic Sudoku'
**When:** User clicks Start button
**Given:** User has selected Classic Sudoku mode


**Then:** Confirmation modal appears with warning message about progress loss
**When:** User clicks Change Mode button
**Given:** User is playing game in Classic Sudoku mode


**Then:** Modal closes, game continues in current mode, no state is reset
**When:** User clicks Cancel button
**Given:** Confirmation modal is displayed


**Then:** Game state resets, returns to mode_selection screen, all modes are unselected
**When:** User clicks 'Change Mode' button in modal
**Given:** Confirmation modal is displayed


**Then:** Game returns to mode_selection screen
**When:** User clicks 'Change Mode' button
**Given:** Game Over screen is displayed


**Then:** mode_selection screen is displayed FIRST, gameplay screen is NOT visible, no mode is pre-selected
**When:** Game loads
**Given:** User opens game for first time


**Then:** Start button is disabled, no mode is highlighted
**When:** User has not clicked any mode card
**Given:** mode_selection screen is displayed


**Then:** Selected mode card is highlighted with selected style, Start button becomes enabled
**When:** User clicks a mode card
**Given:** mode_selection screen is displayed


**Then:** Game transitions to gameplay screen, initializes with selected mode's config, mode_indicator shows active mode
**When:** User clicks Start button
**Given:** User has selected a mode


**Then:** Confirmation modal appears with warning about progress loss
**When:** User clicks Change Mode button
**Given:** User is playing game in selected mode


**Then:** Modal closes, game continues in current mode
**When:** User clicks Cancel button
**Given:** Confirmation modal is displayed


**Then:** Game state resets, returns to mode_selection screen, all modes are unselected
**When:** User clicks 'Change Mode' button in modal
**Given:** Confirmation modal is displayed


**Then:** Game returns to mode_selection screen
**When:** User clicks 'Change Mode' button
**Given:** Game Over screen is displayed


**Then:** A valid move updates the 9x9 grid, move_count increments, and win condition is checked
**When:** Player places a digit
**Given:** 9x9 Sudoku board with initial puzzle


**Then:** game.status = solved, show victory message
**When:** Win condition check occurs
**Given:** Board is correctly completed


**Then:** move_count updates with each valid move
**When:** Player places digits
**Given:** move_count increments accordingly


**Then:** Board restored to previous state, move_count decrements
**When:** Undo action executed
**Given:** Player requests undo


## End Conditions Duplication Note

The end_conditions section intentionally keeps win/lose criteria aligned with Sudoku completion; no AI race end conditions exist in this single-player design.

## Generation Mandatory Requirements

- MUST define at least one mode in available_modes array
- default_mode MUST reference a valid mode identifier from available_modes
- Each mode MUST have: mode (unique id), name, player_mode, players, description
- MUST implement mode_selection screen as FIRST screen shown when game loads
- mode_selection screen is MANDATORY and CANNOT be skipped - no direct access to gameplay
- Each mode card MUST show: name, description, player count, and config summary
- MUST transition to gameplay screen ONLY after mode is selected and Start is clicked
- MUST initialize game with selected mode's configuration from available_modes[selected_index].config
- MUST display current_mode indicator prominently during gameplay
- Changing mode MUST show confirmation modal, then reset game state and return to mode_selection
- MUST implement mode_selection screen as FIRST screen shown to user
- mode_selection screen is MANDATORY and CANNOT be skipped
- mode_selection screen MUST display ALL modes from available_modes array
- MUST show mode icon, name, description, and config for each mode
- Start button MUST be disabled until user selects a mode
- MUST transition to gameplay screen ONLY after mode selected and Start clicked
- MUST initialize game with selected mode's configuration (from available_modes[selected].config)
- MUST display current_mode indicator during gameplay showing active mode name
- MUST provide mode_switcher button accessible during gameplay
- Changing mode MUST show confirmation modal warning about progress loss
- Confirming mode change MUST reset game state and return to mode_selection screen
- Game Over screen MUST provide option to change mode (return to mode_selection)
- MUST set players object to match actual player count for each mode
- MUST define ALL entities listed in state.entities array
- MUST implement ALL actions listed in turns.actions array
- MUST include complete move validation logic
- MUST define clear win/lose conditions in end_conditions
- Core mechanic MUST be accurately described and implemented in mechanics
- Game MUST be fully playable based on specification alone
- All entity properties MUST have initial_state values defined
- default_mode
- player_modes
- common_requirements
- critical_architecture_notes

## Meta

**Game Name:** string
**Game Type:** puzzle
**Default Mode:** classic
**Core Mechanic:** Solve Sudoku puzzles by filling 9x9 grid so each row, column, and 3x3 box contains digits 1-9 exactly once
**Available Modes:**
  **Mode:** classic
  **Name:** Classic Sudoku
  **Config:**
    **Move Limit:** None
    **Time Limit:** None
    **Undo Allowed:** True
    **Hints Available:** 3
  **Players:**
    **Ai:** 0
    **Human:** 1
  **Description:** Solve Sudoku at your own pace
  **Player Mode:** single_player
**Session Minutes:**
  - 5
  - 30
**Supports Multiple Modes:** False