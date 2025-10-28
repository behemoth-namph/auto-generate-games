🎯 Universal Turn-Based Game Requirements Prompt (v5 — Full Version)
Purpose
Produce complete, deterministic, testable, and modular requirements for a turn-based or card-based game engine with a clean, responsive, and accessible UI architecture.
All gameplay logic MUST remain deterministic, replayable, and isolated from UI presentation.
🔴 0. Meta Rules (CORE)
Use normative language (MUST / SHOULD / MAY).
If a feature does not exist, explicitly state:
“This game does not use {feature}.”
All game logic MUST be deterministic, replayable, and free of deadlocks under fixed seeds.
UI and gameplay MUST be decoupled:
/engine/ → logic & state
/ui/ or /components/ → presentation & input only
Replays under identical inputs MUST yield identical outcomes.
🔴 1. Scope & Core Feature Declaration (CORE)
Feature Allowed Values  Description
Board/Socket System present / not used  Discrete spatial playfield where placement occurs
Holdings    present / not used  Player’s hand, deck, or inventory system
Allegiance  allies / enemies / neutral / none   Defines team or side relationships
Turn Mode   sequential / simultaneous   Determines player action order
Randomness  deterministic / pure RNG / none RNG control and seed requirements
Physics Context static / 2D-plane / gravity / directional-flow  Defines spatial or movement physics
✅ Core Assertion:
Every field above MUST be defined. Undefined systems are illegal.
✅ UI Constraint:
UI reads only from selectors — no gameplay logic duplicated in components.
🔴 2. Core Mechanics & Action Legality (CORE)
All player actions MUST specify:
Field   Description
Preconditions   State requirements before action
Legal Targets   Allowed target filters (e.g., ally, enemy, zone)
Postconditions  Resulting state transitions
Illegal Cases   Explicit rejection and feedback behavior
Attempting an illegal action MUST yield immediate rejection: { errorCode, message }.
✅ Determinism Check:
No outcome may depend on hidden or external state.
✅ Deadlock Prevention:
If no legal actions exist, the game MUST auto-pass or trigger draw-check.
✅ UI Hook:
Selectors expose legal moves; components only emit playMove, endTurn, etc.
🔴 3. Board & Spatial Context (CORE)
If used, the board defines legal spatial relationships.
Each board MUST include:
id, coord, tags, occupantId|null, visibility
Board Texture Definition
type: {BOARD_TEXTURE_TYPE}  # 2D static grid, 3D plane, lane map, zone network
physicsMode: {static | gravity | directional-flow | free-plane}
Movement & Obstacles
Define:
movement type: discrete / continuous / turn-gated
blocking zones or forbidden regions
directional or gravitational flow if applicable
If unused:
“This game does not use a spatial board. All actions are abstract.”
✅ Deadlock Rule:
If no entity can move or act due to board saturation, trigger end-check or draw.
🔴 4. Placement & Target Domains (CORE)
Placement Rules
Each placeable object MUST define:
Field   Example
placementType   "unit", "trap", "building", "spell-area"
allowedZones    Named legal zones or coordinates
restrictions    Cannot overlap, cannot spawn on enemy side
visibility  public / private / delayed reveal
Target Rules
Each ability or effect MUST define:
Field   Example
targetType  unit / card / socket / player / random / none
allowedTargets  enemy units, ally minions, empty sockets
illegalTargets  hidden cards, immune heroes
constraints visibility, adjacency, or range limits
✅ Interface Rule:
Legal targets MUST be visually indicated. Illegal attempts return { errorCode, message }.
✅ Determinism Rule:
All targeting outcomes MUST be replayable from log state.
🔴 5. Turn Sequencer & Timing (CORE)
Define clear structure for turn mode and phase transitions.
Order   Phase Name  Actor(s)    Mode    Entry Conditions    Legal Options   DecisionWindowMs    Timeout Policy  Triggers Checked    RNG Use Outcome
1   Preparation Player(s)   sequential/simultaneous Start of round  Arrange holdings, select targets    60000   AutoPass    onStartTurn none    → Action
2   Action  Player(s)   sequential/simultaneous Preparation complete    UseAbility, PlaceUnit, EndTurn  30000   PassTurn    onAction    per ability → End
3   EndOfTurn   Player(s)   sequential/simultaneous Action complete EndTurn 0   none    onEndTurn   none    next actor/round
✅ Determinism Rule:
Turn order and resolution priority MUST be fixed.
✅ UI Hook:
Expose current phase, countdown timer, and active player via selectors.
🔴 6. Win / Lose / Draw Conditions (CORE)
MUST define explicit, deterministic conditions:
Win: Opponent HP ≤ 0 or objectives completed
Lose: All units destroyed or timeout
Draw: No legal moves or simultaneous defeat
✅ End Check:
Executed at end of turn or on immediate trigger.
✅ Mutual Exclusivity:
No overlapping win/lose conditions permitted.
✅ UI Indicator:
Display victory/defeat modals via state events — no logic inside components.
🟡 7. Randomness & Replay Integrity (IMPORTANT)
Field   Example
PRNG_ALGO   xoshiro256**, MersenneTwister
SEED_SOURCE matchId + timestamp or server seed
Rules:
All random outputs MUST be logged.
Replay MUST reproduce identical results under the same seed.
No hidden or client-only randomness.
🟡 8. Player Holdings (IMPORTANT)
If holdings exist:
Player’s hand MUST visibly show all items (face-up, hidden, spent).
Legal actions: draw, discard, reveal, play, shuffle.
Every action MUST emit a logged event.
If not used:
“This game does not use player holdings.”
🟢 9. Entities & Allegiance (OPTIONAL)
Entities define:
unitId, ownerId, allegiance, stats, position
If allegiances used:
isAlly(a,b) and isEnemy(a,b) MUST be deterministic.
Define legality of friendly fire or healing.
If unused:
“No allegiance system; all entities are neutral.”
🟢 10. Interface Visibility & Indicators (OPTIONAL)
UI MUST include visible indicators for:
Current turn owner
Next turn order
Timer countdown
Phase indicator
Stats: cards remaining, units, round, score
✅ Placeholder Rule:
Visual layout MAY be deferred, but placeholder hooks MUST exist.
🟢 11. Multiplayer Readiness (OPTIONAL)
Command schema:
{
  "matchId": "...",
  "turnId": "...",
  "actorId": "...",
  "commandId": "...",
  "params": {},
  "prevStateHash": "..."
}
Commands MUST be ordered and idempotent.
Reserved hooks: onConnect, onCommand, onState, onTimeout, onDesync.
🟢 12. Event Bus (OPTIONAL)
Emit structured events for replay and UI synchronization:
evt.TurnStart, evt.ActionValidated, evt.UnitMoved,
evt.Damaged, evt.StatusApplied, evt.Win, evt.Lose, evt.Draw,
ui.layout.change, ui.accessibility.announce
Event payload:
{ "name": "evt.TurnStart", "time": 12345, "actorId": "p1", "entities": [], "rngTrace": [] }
🔴 13. Logging & Replay Requirements (CORE)
All actions, RNG outputs, and results MUST be logged:
{ turnId, actorId, seed, commandTrace }
Replays MUST reproduce identical transitions.
UI MAY subscribe to events but MUST NOT mutate state.
🧩 Phase 1 — Refactor for Modifiability (UI Architecture Rules)
Goals
Preserve gameplay logic and rules.
Restructure UI for flexible layout and accessibility.
Make code componentized, testable, and alignment-friendly.
Requirements
1. Project Hygiene
Add or update a minimal README.md with build/run steps.
Introduce /ui/ or /components/ folder and design-tokens.(ts|js|json|css) file.
Add constants/layout.(ts|js) for header/footer heights, tap-target size, safe-area padding, and aspect ratios.
2. State Selectors
Create selectors exposing:
currentPlayer, isMyTurn, turnNumber, timeLeftSec, scores, entityCounts, resources, recentMoves.
UI reads only from selectors; do not recompute rules in components.
3. Component Boundaries
Split monolithic screens into:
TurnBar
GameArea (board/canvas/SVG)
ActionBar
InfoDrawer (collapsible)
ModalHistory
Each component:
Receives plain props from selectors.
Emits events: endTurn, undo, playMove(entityId).
4. Responsive Containers
Root: full viewport height → height: 100svh.
Board wrapper: maintain aspect-ratio; fill remaining space.
Use sticky header/footer and respect env(safe-area-inset-*).
5. SVG/Canvas Hit-Testing
Keep rendering intact but ensure ≥ 40×40 px tap targets using transparent hit areas (fill="transparent", pointer-events="all").
6. Accessibility
All interactive cells MUST have aria-label, e.g. “Pit 5, 3 stones; tap to play.”
Use aria-live="polite" for turn changes and countdowns.
Keyboard focus states visible; focus order matches visual order.
7. Deliverables
A single PR or patch containing:
New components + selectors (no gameplay changes).
Updated imports/wiring using selectors.
README section explaining layout knobs.
8. Acceptance Criteria
Build passes; gameplay identical.
Core view mounts with header/board/footer skeleton.
All interactive elements accessible; board fully playable.
📱 Additional UI Acceptance Summary (Integrated)
Category    Requirements
Determinism Replay-safe, seeded RNG
UI Structure    5 core components + selectors layer
Responsiveness  100 svh + safe-area support
Accessibility   ARIA labels + keyboard focus + ≥ 40 px hit-targets
Maintainability Design tokens + layout constants
Testability Logic isolated from UI
Replay Integrity    Identical outcome under identical logs
🧭 Markdown Usage
Markdown Element    Purpose
#, ##, ###  Hierarchical priority (CORE > IMPORTANT > OPTIONAL)
Bold    Normative language emphasis
Tables  Structured enumerations
Code blocks Preserve schema & JSON/YAML
📱 icon   Layout / Accessibility rules
✅ End of v5 — Full Prompt
You can now copy-paste this block directly into your universal game generator to produce deterministic, modular, and mobile-alignment-ready codebases.