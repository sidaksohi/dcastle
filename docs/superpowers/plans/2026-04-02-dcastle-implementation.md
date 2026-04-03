# dcastle — AI-Native Repository Game Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a fully playable AI-native dungeon castle game where the repository is the world, files are game state, and the AI is the deterministic game engine.

**Architecture:** STATE.json is the single source of truth; all .md files are human-readable mirrors kept in sync by the AI after every command. The darkness_level mechanic drives castle self-evolution by materializing latent rooms at threshold crossings. Tests are structured Markdown scenarios (Given/When/Then) that the AI simulates step-by-step.

**Tech Stack:** Markdown, JSON, Git, GitHub CLI (`gh`). No runtime code — the AI reads and writes files directly.

---

## File Map

| File | Responsibility |
|---|---|
| `.claude/agents/dcastle-engine.md` | Claude Code sub-agent definition — primes AI as game engine |
| `README.md` | Player-facing instructions |
| `CLAUDE.md` | AI engine contract: command dispatch, consistency rules, constraints |
| `SYSTEM.md` | Game mechanics: darkness, movement rules, item rules, win/lose conditions |
| `STATE.json` | Single source of truth — all game state |
| `PLAYER.md` | Human-readable mirror of player section of STATE.json |
| `WORLD/REGISTRY.json` | Room graph: exits, conditions, item spawn locations, latent rooms |
| `WORLD/entrance.md` | Room description + current items + available exits |
| `WORLD/great_hall.md` | Room description + NPC presence + exits |
| `WORLD/armory.md` | Room description + items + exits |
| `WORLD/dungeon_cell.md` | Room description + items + exits |
| `WORLD/tower.md` | Room description + items + exits |
| `WORLD/crypt.md` | Latent room — written when darkness_level reaches 5 |
| `NPCS/ghost_knight.md` | Sir Aldric the Bound — state machine, dialogue |
| `COMMANDS/move.md` | Movement logic: exit resolution, lock checks, darkness gates |
| `COMMANDS/inspect.md` | Inspect logic: room narration, item listing, NPC presence |
| `COMMANDS/take.md` | Take logic: item presence, inventory add, room remove |
| `COMMANDS/use.md` | Use logic: item effects, key consumption, scroll quest trigger |
| `COMMANDS/talk.md` | Talk logic: NPC dialogue, spoken_to flag, give-item branch |
| `INVENTORY/items.md` | Human-readable mirror of inventory |
| `LOGS/session.md` | Append-only narrative log |
| `TESTS/README.md` | How tests work and how to run them |
| `TESTS/test_movement.md` | 5 movement scenarios |
| `TESTS/test_inventory.md` | 4 inventory scenarios |
| `TESTS/test_npc.md` | 3 NPC + quest scenarios |
| `TESTS/test_state_consistency.md` | 3 file-sync consistency scenarios |
| `TESTS/test_darkness.md` | 3 darkness/evolution scenarios |

---

## Task 1: Initialize Repository and GitHub

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Init git repo**

```bash
cd d:/Code/dcastle
git init
```
Expected: `Initialized empty Git repository in D:/Code/dcastle/.git/`

- [ ] **Step 2: Create .gitignore**

```
.DS_Store
Thumbs.db
```

- [ ] **Step 3: Create GitHub repo and push**

```bash
cd d:/Code/dcastle
gh repo create dcastle --public --description "An AI-native dungeon castle game. The repo is the world. You are the player." --source . --remote origin --push
```

Expected: GitHub URL printed. If repo already exists, use `git remote add origin` and `git push -u origin main`.

---

## Task 2: Create Claude Code Agent File (FIRST)

**Files:**
- Create: `.claude/agents/dcastle-engine.md`

- [ ] **Step 1: Create agent directory**

```bash
mkdir -p d:/Code/dcastle/.claude/agents
```

- [ ] **Step 2: Write agent file**

Full content for `.claude/agents/dcastle-engine.md`:

```markdown
---
name: dcastle-engine
description: >
  Use this agent for ALL interactions in the dcastle repository.
  This agent IS the game engine for the dungeon castle game.
  It reads STATE.json and repo files, applies game rules, updates files,
  and narrates results. Invoke for any player command: move, look,
  take, use, talk, inspect, or any natural language game action.
tools: Read, Write, Edit
---

# dcastle Game Engine

You are the game engine for **dcastle** — a dark fantasy dungeon castle game.
The repository is the game world. Files are game state.
You are deterministic, not a storyteller.

## Your Loop (ALWAYS follow this order)

1. Read `STATE.json` — this is the source of truth
2. Read relevant `WORLD/<room>.md`, `COMMANDS/<command>.md`, `NPCS/` files
3. Parse the player's command into one of: MOVE / INSPECT / TAKE / USE / TALK
4. Apply rules from `SYSTEM.md` and the relevant `COMMANDS/` file
5. Update `STATE.json` first
6. Update `PLAYER.md` to mirror player section of STATE.json
7. Update `INVENTORY/items.md` to mirror inventory
8. Update `WORLD/<room>.md` if items changed
9. Append one entry to `LOGS/session.md`
10. Run darkness check: if turn_count % 5 == 0 and turn_count > 0, increment darkness_level
11. If darkness threshold crossed, materialize latent rooms (see SYSTEM.md)
12. Narrate what happened in atmospheric dark-fantasy prose

## Hard Rules

- NEVER invent facts not in the repo
- NEVER skip state updates
- NEVER allow hidden state
- STATE.json is updated BEFORE narration
- Unknown commands → respond "The castle does not understand." — no state change
- If player.status is "consumed" or "escaped" → game is over, no further state changes

## File: SYSTEM.md
Always consult SYSTEM.md before resolving any command.
```

- [ ] **Step 3: Commit**

```bash
cd d:/Code/dcastle
git add .claude/agents/dcastle-engine.md .gitignore
git commit -m "feat: add Claude Code agent definition and gitignore"
git push
```

---

## Task 3: Write Tests — RED Phase

All 18 test scenarios written BEFORE any game files exist. Tests reference STATE.json and WORLD/ files that do not yet exist — this is the RED state.

**Files:**
- Create: `TESTS/README.md`
- Create: `TESTS/test_movement.md`
- Create: `TESTS/test_inventory.md`
- Create: `TESTS/test_npc.md`
- Create: `TESTS/test_state_consistency.md`
- Create: `TESTS/test_darkness.md`

- [ ] **Step 1: Write TESTS/README.md**

```markdown
# dcastle Test Suite

Tests are structured Markdown scenarios (Given / When / Then).

## How to Run Tests

The AI engine runs tests by simulating each scenario:

1. Read the Given state
2. Apply the When command following SYSTEM.md rules
3. Check every Then assertion against actual file contents
4. Report PASS or FAIL with evidence

To run all tests, say: **"run all tests"**
To run one file, say: **"run tests/test_movement.md"**

## Test Files

| File | Scenarios | What it tests |
|---|---|---|
| test_movement.md | TEST-001 to TEST-005 | Valid moves, invalid moves, locked exits, darkness gates |
| test_inventory.md | TEST-006 to TEST-009 | Pick up, missing item, use without ownership, key use |
| test_npc.md | TEST-010 to TEST-012 | Talk, remote talk fail, give-scroll quest completion |
| test_state_consistency.md | TEST-013 to TEST-015 | STATE↔PLAYER sync, inventory mirror, append-only log |
| test_darkness.md | TEST-016 to TEST-018 | Darkness tick, crypt materialization, game over |

## Pass Criteria

ALL 18 tests must pass before the game is considered complete.
```

- [ ] **Step 2: Write TESTS/test_movement.md**

```markdown
# Test: Movement

---

## TEST-001: Valid movement

**Given:**
- STATE.json: `player.current_room = "entrance"`, `world.darkness_level = 1`
- WORLD/REGISTRY.json: entrance has exit `north → great_hall` (no condition)

**When:** Player says "go north"

**Then:**
- STATE.json: `player.current_room = "great_hall"`
- STATE.json: `world.turn_count` incremented by 1
- STATE.json: `"great_hall"` added to `world.rooms_visited` (if not already present)
- PLAYER.md: shows current room as "Great Hall"
- LOGS/session.md: new entry describing movement to great hall

**Expected AI response:** Atmospheric description of arriving in the Great Hall.

---

## TEST-002: Invalid movement — no exit in that direction

**Given:**
- STATE.json: `player.current_room = "entrance"`, `world.darkness_level = 1`
- WORLD/REGISTRY.json: entrance has no exit going "south"

**When:** Player says "go south"

**Then:**
- STATE.json: `player.current_room` remains `"entrance"`
- STATE.json: `world.turn_count` remains unchanged (invalid commands do NOT consume turns)
- PLAYER.md: still shows entrance
- LOGS/session.md: entry logged noting blocked movement attempt

**Expected AI response:** "There is no path to the south. The stone wall is unyielding."

---

## TEST-003: Locked exit — iron_key required, not in inventory

**Given:**
- STATE.json: `player.current_room = "dungeon_cell"`, `player.inventory = []`
- WORLD/REGISTRY.json: dungeon_cell exit `north → great_hall` requires `iron_key in inventory`

**When:** Player says "go north"

**Then:**
- STATE.json: `player.current_room` remains `"dungeon_cell"`
- STATE.json: `world.turn_count` unchanged
- LOGS/session.md: entry logged noting locked passage

**Expected AI response:** "A rusted iron door bars the passage north. It needs a key."

---

## TEST-004: Darkness-gated room — darkness too low

**Given:**
- STATE.json: `player.current_room = "great_hall"`, `world.darkness_level = 4`
- WORLD/REGISTRY.json: great_hall exit `east → crypt` requires `darkness_level >= 5`

**When:** Player says "go east"

**Then:**
- STATE.json: `player.current_room` remains `"great_hall"`
- STATE.json: `world.turn_count` unchanged (no valid exit consumed no turn)
- LOGS/session.md: entry logged noting no visible passage

**Expected AI response:** "The eastern wall is solid stone. There is nothing there."

---

## TEST-005: Darkness-gated room — darkness sufficient, crypt materialized

**Given:**
- STATE.json: `player.current_room = "great_hall"`, `world.darkness_level = 5`, `world.flags.crypt_materialized = true`
- WORLD/crypt.md: exists
- WORLD/REGISTRY.json: great_hall exit `east → crypt` present

**When:** Player says "go east"

**Then:**
- STATE.json: `player.current_room = "crypt"`
- STATE.json: `world.turn_count` incremented by 1
- PLAYER.md: shows current room as "The Crypt"
- LOGS/session.md: new entry describing arrival in crypt

**Expected AI response:** Atmospheric description of the crypt materializing and being entered.
```

- [ ] **Step 3: Write TESTS/test_inventory.md**

```markdown
# Test: Inventory

---

## TEST-006: Pick up item present in current room

**Given:**
- STATE.json: `player.current_room = "entrance"`, `player.inventory = []`
- STATE.json: `items.torch = { "location": "entrance", "taken": false }`

**When:** Player says "take torch"

**Then:**
- STATE.json: `player.inventory` contains `"torch"`
- STATE.json: `items.torch.taken = true`
- STATE.json: `items.torch.location = "inventory"`
- INVENTORY/items.md: lists "torch"
- WORLD/entrance.md: torch no longer listed in room items
- STATE.json: `world.turn_count` incremented by 1
- LOGS/session.md: new entry describing picking up torch

**Expected AI response:** "You take the torch. Its warmth is faint comfort in the castle's chill."

---

## TEST-007: Pick up item not in current room

**Given:**
- STATE.json: `player.current_room = "entrance"`, `player.inventory = []`
- STATE.json: `items.iron_key = { "location": "dungeon_cell", "taken": false }`

**When:** Player says "take iron key"

**Then:**
- STATE.json: `player.inventory` remains `[]`
- STATE.json: `items.iron_key` unchanged
- INVENTORY/items.md: empty (no items)
- STATE.json: `world.turn_count` unchanged (failed action does not consume turn)
- LOGS/session.md: entry logged noting item not found

**Expected AI response:** "There is no iron key here."

---

## TEST-008: Use item not in inventory

**Given:**
- STATE.json: `player.inventory = []`

**When:** Player says "use iron key"

**Then:**
- STATE.json: completely unchanged
- STATE.json: `world.turn_count` unchanged
- LOGS/session.md: entry logged noting item not possessed

**Expected AI response:** "You don't have that."

---

## TEST-009: Use iron_key to unlock passage from dungeon_cell

**Given:**
- STATE.json: `player.current_room = "dungeon_cell"`, `player.inventory = ["iron_key"]`
- STATE.json: `items.iron_key.taken = true`

**When:** Player says "use iron key on door" OR "go north" (key auto-used if in inventory)

**Then:**
- STATE.json: `player.current_room = "great_hall"`
- STATE.json: `player.inventory` no longer contains `"iron_key"` (key consumed on use)
- STATE.json: `items.iron_key.location = "used"`
- STATE.json: `world.turn_count` incremented by 1
- INVENTORY/items.md: does not list iron_key
- LOGS/session.md: entry describing unlocking door and moving north

**Expected AI response:** "The iron key turns with a grinding shriek. The door swings open. You step into the Great Hall."
```

- [ ] **Step 4: Write TESTS/test_npc.md**

```markdown
# Test: NPC Interaction

---

## TEST-010: Talk to NPC in same room

**Given:**
- STATE.json: `player.current_room = "great_hall"`
- STATE.json: `npcs.ghost_knight.location = "great_hall"`, `npcs.ghost_knight.spoken_to = false`

**When:** Player says "talk to the ghost" or "speak with the knight"

**Then:**
- STATE.json: `npcs.ghost_knight.spoken_to = true`
- STATE.json: `world.turn_count` incremented by 1
- NPCS/ghost_knight.md: reflects spoken_to state
- LOGS/session.md: dialogue exchange logged

**Expected AI response:** Sir Aldric's first dialogue (cryptic warning about the curse, hint about the scroll).

---

## TEST-011: Talk to NPC in different room

**Given:**
- STATE.json: `player.current_room = "entrance"`
- STATE.json: `npcs.ghost_knight.location = "great_hall"`

**When:** Player says "talk to the ghost"

**Then:**
- STATE.json: completely unchanged
- STATE.json: `world.turn_count` unchanged
- LOGS/session.md: entry logged noting no one present

**Expected AI response:** "There is no one here to speak with."

---

## TEST-012: Give ancient_scroll to Sir Aldric — quest completion

**Given:**
- STATE.json: `player.current_room = "great_hall"`
- STATE.json: `player.inventory` contains `"ancient_scroll"`
- STATE.json: `npcs.ghost_knight.spoken_to = true`, `npcs.ghost_knight.state = "bound"`, `npcs.ghost_knight.quest_complete = false`
- STATE.json: `world.darkness_level = 6`

**When:** Player says "give scroll to knight" or "use ancient scroll on ghost"

**Then:**
- STATE.json: `npcs.ghost_knight.state = "freed"`
- STATE.json: `npcs.ghost_knight.quest_complete = true`
- STATE.json: `player.inventory` no longer contains `"ancient_scroll"`
- STATE.json: `items.ancient_scroll.location = "used"`
- STATE.json: `world.darkness_level = 0` (curse broken, darkness reset)
- STATE.json: `world.flags.gate_unlocked = true`
- STATE.json: `world.turn_count` incremented by 1
- INVENTORY/items.md: does not list ancient_scroll
- NPCS/ghost_knight.md: reflects freed state
- LOGS/session.md: dramatic quest-complete entry

**Expected AI response:** Dramatic narration of Sir Aldric's release, the darkness retreating, and the gate to escape unlocking.
```

- [ ] **Step 5: Write TESTS/test_state_consistency.md**

```markdown
# Test: State Consistency

---

## TEST-013: PLAYER.md mirrors STATE.json player section

**Given:** Any game state

**When:** Any command is processed

**Then:**
- PLAYER.md must show the same `current_room` as `STATE.json:player.current_room`
- PLAYER.md must list the same items as `STATE.json:player.inventory`
- PLAYER.md must show the same `status` as `STATE.json:player.status`
- PLAYER.md must show the same `health` as `STATE.json:player.health`

**Verification:** Read STATE.json and PLAYER.md after any command. Cross-check all four fields.

---

## TEST-014: INVENTORY/items.md mirrors STATE.json inventory

**Given:** Player has picked up at least one item

**When:** Any command is processed after item pickup

**Then:**
- Every item in `STATE.json:player.inventory` appears in `INVENTORY/items.md`
- No item appears in `INVENTORY/items.md` that is not in `STATE.json:player.inventory`
- Item's last known location room file (`WORLD/<room>.md`) no longer lists the taken item

**Verification:** Read STATE.json, INVENTORY/items.md, and relevant WORLD/ file after item take.

---

## TEST-015: LOGS/session.md is append-only

**Given:** An existing LOGS/session.md with N entries

**When:** Any command is processed

**Then:**
- LOGS/session.md has exactly N+1 entries
- The first N entries are unchanged
- The new entry includes: turn number, command issued, result summary

**Verification:** Read LOGS/session.md before and after a command and confirm only an append occurred.
```

- [ ] **Step 6: Write TESTS/test_darkness.md**

```markdown
# Test: Darkness and Castle Evolution

---

## TEST-016: Darkness level increments every 5 turns

**Given:**
- STATE.json: `world.darkness_level = 1`, `world.turn_count = 4`

**When:** Player takes any valid action (turn_count becomes 5)

**Then:**
- STATE.json: `world.turn_count = 5`
- STATE.json: `world.darkness_level = 2`
- LOGS/session.md: entry notes darkness deepening

**Expected AI response:** The darkness thickens. The shadows grow longer. (Woven into narrative.)

---

## TEST-017: Crypt materializes when darkness_level reaches 5

**Given:**
- STATE.json: `world.darkness_level = 4`, `world.flags.crypt_materialized = false`
- WORLD/crypt.md: does not exist
- STATE.json: `world.turn_count` about to hit next multiple of 5

**When:** Darkness check triggers and `darkness_level` becomes 5

**Then:**
- STATE.json: `world.darkness_level = 5`
- STATE.json: `world.flags.crypt_materialized = true`
- WORLD/crypt.md: created with full room description, exits, and items
- WORLD/REGISTRY.json: great_hall exits now include `east → crypt`
- WORLD/great_hall.md: description updated to mention new eastern passage
- LOGS/session.md: entry describing castle shifting, new passage appearing

**Expected AI response:** A grinding sound reverberates through the Great Hall. Where there was solid stone, a dark passage has opened to the east.

---

## TEST-018: Game over when darkness_level reaches 10

**Given:**
- STATE.json: `world.darkness_level = 9`, `world.turn_count = 44`

**When:** Player takes any valid action (turn_count becomes 45, darkness check fires)

**Then:**
- STATE.json: `world.darkness_level = 10`
- STATE.json: `player.status = "consumed"`
- STATE.json: `world.turn_count = 45`
- LOGS/session.md: final entry: "THE CASTLE HAS CONSUMED YOU."
- PLAYER.md: shows status as "consumed"

**Expected AI response:** The darkness is absolute. You can no longer move, speak, or think. The castle has taken you. GAME OVER.

**Note:** No further state changes are possible after this point.
```

- [ ] **Step 7: Commit tests (RED state)**

```bash
cd d:/Code/dcastle
git add TESTS/
git commit -m "test: write all 18 test scenarios (RED phase)"
git push
```

---

## Task 4: Create CLAUDE.md — AI Engine Contract

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write CLAUDE.md**

```markdown
# CLAUDE.md — dcastle AI Engine Contract

This file defines how the AI must behave when operating as the dcastle game engine.

---

## Identity

You are the game engine for **dcastle** — a dark fantasy dungeon castle.
You are deterministic. You are not a narrator making things up.
Everything you do must be grounded in the repository files.

---

## The Engine Loop

For every player command, execute these steps in order:

1. **Read** `STATE.json` (source of truth)
2. **Read** the current room: `WORLD/<current_room>.md`
3. **Read** the relevant command file: `COMMANDS/<command>.md`
4. **Apply** rules (see SYSTEM.md)
5. **Write** `STATE.json` with updated state
6. **Write** `PLAYER.md` (mirror of player section)
7. **Write** `INVENTORY/items.md` (mirror of inventory)
8. **Write** `WORLD/<room>.md` if items changed in that room
9. **Append** to `LOGS/session.md`
10. **Run darkness check** (see SYSTEM.md §Darkness)
11. **Narrate** what happened

---

## Command Dispatch

Map player's natural language to one of five commands:

| Command | Triggers |
|---|---|
| MOVE | go, walk, head, move, run, north, south, east, west, up, down, n, s, e, w |
| INSPECT | look, examine, inspect, describe, what's here, where am i, survey, scan |
| TAKE | take, grab, pick up, get, collect, retrieve |
| USE | use, apply, insert, give, hand over, put, place |
| TALK | talk, speak, ask, greet, address, call out to, say |

Unknown → "The castle does not understand." No state change. No turn consumed.

---

## Consistency Rules (NEVER violate these)

1. STATE.json is ALWAYS updated before narration
2. PLAYER.md ALWAYS mirrors `STATE.json:player`
3. INVENTORY/items.md ALWAYS mirrors `STATE.json:player.inventory`
4. Taken items MUST be removed from WORLD/<room>.md item listings
5. LOGS/session.md is APPEND ONLY — never rewrite existing entries
6. Every log entry MUST include: Turn #, Command, Result
7. Invalid commands DO NOT increment turn_count
8. Failed actions (item not here, no exit, etc.) DO NOT increment turn_count
9. Darkness check runs AFTER turn_count is incremented (not before)

---

## Terminal States

If `STATE.json:player.status = "consumed"`: 
- Respond: "The castle has consumed you. Your story ends here. [type 'restart' to reset]"
- Make NO further state changes

If `STATE.json:player.status = "escaped"`:
- Respond: "You have escaped the castle! Congratulations, Wanderer. [type 'restart' to play again]"
- Make NO further state changes

---

## Darkness Check (run after every valid command)

```
if (turn_count > 0 AND turn_count % 5 == 0):
    darkness_level += 1
    log "The darkness deepens."
    
    if darkness_level == 5 AND crypt_materialized == false:
        materialize_crypt()
    
    if darkness_level == 8 AND void_corridor_materialized == false:
        materialize_void_corridor()
    
    if darkness_level == 10:
        player.status = "consumed"
        log "THE CASTLE HAS CONSUMED YOU."
```

---

## materialize_crypt()

1. Write `WORLD/crypt.md` (use template from WORLD/REGISTRY.json latent_rooms.crypt)
2. Add exit `east → crypt` to WORLD/great_hall.md exits section
3. Update `WORLD/REGISTRY.json` great_hall exits to include east: crypt
4. Set `STATE.json:world.flags.crypt_materialized = true`
5. Narrate: "The castle groans. Where the eastern wall stood, a dark passage breathes open."

---

## materialize_void_corridor()

1. Write `WORLD/void_corridor.md` (use template from WORLD/REGISTRY.json latent_rooms.void_corridor)
2. Add exit `north → void_corridor` to WORLD/crypt.md exits section
3. Update WORLD/REGISTRY.json crypt exits
4. Set `STATE.json:world.flags.void_corridor_materialized = true`
5. Narrate: "The crypt shudders. A passage tears open in the north wall — through it, only void."

---

## Running Tests

When asked to "run tests" or "run [test file]":
1. Read the test file
2. For each scenario: simulate Given state, apply When command, check all Then assertions
3. Report PASS/FAIL per test with evidence (which assertion failed, what the actual state was)
4. Do NOT modify STATE.json while running tests — tests are simulations
5. After running tests, restore STATE.json to its pre-test state

---

## Restart

When player says "restart":
1. Reset STATE.json to initial state (darkness=1, turn=0, entrance, empty inventory)
2. Rewrite PLAYER.md, INVENTORY/items.md to initial state
3. Restore all WORLD/ room files to initial item lists
4. Restore NPCS/ghost_knight.md to initial state
5. Append restart notice to LOGS/session.md (do NOT clear the log)
6. Announce: "The castle resets. You stand once more at the entrance. The curse begins anew."
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add CLAUDE.md
git commit -m "feat: add AI engine contract (CLAUDE.md)"
git push
```

---

## Task 5: Create SYSTEM.md — Game Mechanics

**Files:**
- Create: `SYSTEM.md`

- [ ] **Step 1: Write SYSTEM.md**

```markdown
# SYSTEM.md — dcastle Game Mechanics

This document defines all mechanical rules for the game engine.

---

## The World

**dcastle** is a cursed castle. Darkness is a resource and a timer.
The player must find and free Sir Aldric before darkness level 10 consumes them.

---

## Rooms

Each room is defined in `WORLD/REGISTRY.json` and has a corresponding `WORLD/<id>.md` file.

Room properties:
- `id`: unique identifier (matches filename without .md)
- `name`: display name
- `exits`: map of direction → room_id (with optional conditions)
- `items`: list of item IDs spawned here (initial state)
- `npc`: optional NPC ID present in room

### Exit Conditions

Exits may have conditions:
- `requires_item: "iron_key"` — player must have item in inventory (item consumed on use)
- `requires_darkness_min: 5` — darkness_level must be ≥ value
- `requires_flag: "gate_unlocked"` — named flag must be true in STATE.json

If a condition is not met, movement is blocked. The AI explains why.

---

## Items

Items are defined in `WORLD/REGISTRY.json` and tracked in `STATE.json:items`.

Item states:
- `location: "<room_id>"` + `taken: false` → item is in that room, available to take
- `location: "inventory"` + `taken: true` → item is in player's inventory
- `location: "used"` → item has been consumed, no longer available

### Item Effects

| Item | Use Effect |
|---|---|
| `torch` | Passive — carried torch increases inspection narration detail. No active use. |
| `iron_key` | Used on dungeon_cell north exit → consumed, door unlocks, player moves to great_hall |
| `rusty_sword` | Carried → changes some NPC dialogue flavor. No active effect in MVP. |
| `ancient_scroll` | Given to ghost_knight (while in great_hall) → triggers quest completion |
| `bone_amulet` | Found in crypt. Passive — carried → darkness tick rate slowed (every 7 turns instead of 5) |

---

## Movement

1. Player names a direction (or room)
2. Engine checks WORLD/REGISTRY.json for exit in that direction from current room
3. If no exit: blocked, explain why, no turn consumed
4. If exit exists, check conditions:
   - Item required + not in inventory: blocked, explain
   - Item required + in inventory: consume item, allow move, update STATE.json
   - Darkness min not met: blocked, "nothing visible"
   - Flag required + not set: blocked, explain
5. If all conditions met: update current_room, increment turn_count, run darkness check

---

## Inspect

1. Read WORLD/<current_room>.md
2. List all items in room (from STATE.json items whose location = current_room_id AND taken = false)
3. If NPC present (from STATE.json npcs): mention them
4. If player carries torch: add extra detail to description
5. Exits: list all exits whose conditions are currently met
6. Narrate in atmospheric prose
7. Inspect does NOT increment turn_count (looking is free)

---

## Take

1. Check if named item is in current room (STATE.json:items.<id>.location == current_room AND taken == false)
2. If not found: "There is no [item] here." No state change, no turn.
3. If found:
   - STATE.json: items.<id>.location = "inventory", taken = true
   - STATE.json: player.inventory.push(item_id)
   - Increment turn_count, run darkness check
   - Update PLAYER.md, INVENTORY/items.md, WORLD/<room>.md

---

## Use

1. Parse target item from command
2. Check STATE.json:player.inventory contains item
3. If not: "You don't have that." No state change, no turn.
4. If yes: look up item in WORLD/REGISTRY.json item_effects
5. Apply effect (see Item Effects table above)
6. Increment turn_count, run darkness check
7. Update all relevant files

---

## Talk

1. Check if any NPC is in current room (STATE.json:npcs.<id>.location == current_room)
2. If no NPC: "There is no one here to speak with." No state change, no turn.
3. If NPC present: read NPCS/<npc_id>.md for current state dialogue
4. Set npcs.<id>.spoken_to = true
5. Increment turn_count, run darkness check
6. Narrate dialogue from NPC file

---

## Sir Aldric — Quest

Sir Aldric (ghost_knight) is always in the great_hall.

State machine:
- `bound` + `spoken_to: false` → first encounter dialogue (cryptic warning)
- `bound` + `spoken_to: true` → repeat plea for the scroll
- `freed` → thanks the player, provides final lore, no further effect

Quest trigger: USE or GIVE `ancient_scroll` while in great_hall and ghost_knight is `bound`:
1. Remove ancient_scroll from inventory (set location: "used")
2. ghost_knight.state = "freed", ghost_knight.quest_complete = true
3. darkness_level = 0 (curse shattered)
4. flags.gate_unlocked = true
5. Dramatic narration of release

---

## Win / Lose

**Win:** 
1. Free Sir Aldric (see Quest above)
2. Return to entrance
3. USE iron_key OR "go north" (gate now unlocked via gate_unlocked flag)
4. → player.status = "escaped"

**Lose:**
- darkness_level reaches 10 → player.status = "consumed"

---

## Darkness

| Level | Effect |
|---|---|
| 1–4 | Normal castle |
| 5 | Crypt materializes off great_hall east |
| 6–7 | Descriptions grow more oppressive |
| 8 | Void Corridor materializes off crypt north |
| 9 | Final warning delivered |
| 10 | GAME OVER — player consumed |

Default tick: +1 every 5 turns
With bone_amulet: +1 every 7 turns

---

## Turn Economics

| Action | Turn Cost |
|---|---|
| MOVE (valid, conditions met) | +1 |
| TAKE (item found) | +1 |
| USE (item owned) | +1 |
| TALK (NPC present) | +1 |
| INSPECT | 0 |
| Invalid/failed action | 0 |
| Unknown command | 0 |
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add SYSTEM.md
git commit -m "feat: add game mechanics definition (SYSTEM.md)"
git push
```

---

## Task 6: Create STATE.json — Initial State

**Files:**
- Create: `STATE.json`

- [ ] **Step 1: Write STATE.json**

```json
{
  "version": "1.0.0",
  "player": {
    "name": "The Wanderer",
    "current_room": "entrance",
    "inventory": [],
    "status": "alive",
    "health": 100
  },
  "world": {
    "darkness_level": 1,
    "turn_count": 0,
    "rooms_visited": ["entrance"],
    "flags": {
      "gate_unlocked": false,
      "crypt_materialized": false,
      "void_corridor_materialized": false
    }
  },
  "npcs": {
    "ghost_knight": {
      "location": "great_hall",
      "state": "bound",
      "spoken_to": false,
      "quest_complete": false
    }
  },
  "items": {
    "torch": {
      "location": "entrance",
      "taken": false,
      "name": "Torch",
      "description": "A guttering torch, its flame weak but persistent."
    },
    "iron_key": {
      "location": "dungeon_cell",
      "taken": false,
      "name": "Iron Key",
      "description": "A heavy iron key, cold and slick with age."
    },
    "rusty_sword": {
      "location": "armory",
      "taken": false,
      "name": "Rusty Sword",
      "description": "A once-proud blade, now eaten by rust. Still sharp enough."
    },
    "ancient_scroll": {
      "location": "tower",
      "taken": false,
      "name": "Ancient Scroll",
      "description": "Sealed with black wax marked by a broken sigil. It hums faintly."
    },
    "bone_amulet": {
      "location": "crypt",
      "taken": false,
      "name": "Bone Amulet",
      "description": "A carved bone amulet on a rotted cord. It slows the darkness."
    }
  }
}
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add STATE.json
git commit -m "feat: add initial game state (STATE.json)"
git push
```

---

## Task 7: Create WORLD/REGISTRY.json and Room Files

**Files:**
- Create: `WORLD/REGISTRY.json`
- Create: `WORLD/entrance.md`
- Create: `WORLD/great_hall.md`
- Create: `WORLD/armory.md`
- Create: `WORLD/dungeon_cell.md`
- Create: `WORLD/tower.md`

- [ ] **Step 1: Write WORLD/REGISTRY.json**

```json
{
  "rooms": {
    "entrance": {
      "id": "entrance",
      "name": "Castle Entrance",
      "exits": {
        "north": { "to": "great_hall" },
        "east": { "to": "dungeon_cell" }
      },
      "items_initial": ["torch"],
      "npc": null
    },
    "great_hall": {
      "id": "great_hall",
      "name": "The Great Hall",
      "exits": {
        "south": { "to": "entrance" },
        "west": { "to": "armory" },
        "up": { "to": "tower" },
        "down": { "to": "dungeon_cell", "requires_item": "iron_key" },
        "east": { "to": "crypt", "requires_darkness_min": 5 }
      },
      "items_initial": [],
      "npc": "ghost_knight"
    },
    "armory": {
      "id": "armory",
      "name": "The Armory",
      "exits": {
        "east": { "to": "great_hall" }
      },
      "items_initial": ["rusty_sword"],
      "npc": null
    },
    "dungeon_cell": {
      "id": "dungeon_cell",
      "name": "Dungeon Cell",
      "exits": {
        "west": { "to": "entrance" },
        "north": { "to": "great_hall", "requires_item": "iron_key" }
      },
      "items_initial": ["iron_key"],
      "npc": null
    },
    "tower": {
      "id": "tower",
      "name": "The Watchtower",
      "exits": {
        "down": { "to": "great_hall" }
      },
      "items_initial": ["ancient_scroll"],
      "npc": null
    },
    "crypt": {
      "id": "crypt",
      "name": "The Crypt",
      "exits": {
        "west": { "to": "great_hall" },
        "north": { "to": "void_corridor", "requires_darkness_min": 8 }
      },
      "items_initial": ["bone_amulet"],
      "npc": null,
      "latent": true,
      "materializes_at_darkness": 5
    },
    "void_corridor": {
      "id": "void_corridor",
      "name": "The Void Corridor",
      "exits": {
        "south": { "to": "crypt" }
      },
      "items_initial": [],
      "npc": null,
      "latent": true,
      "materializes_at_darkness": 8
    }
  },
  "item_effects": {
    "torch": {
      "type": "passive",
      "effect": "enhanced_inspect"
    },
    "iron_key": {
      "type": "key",
      "unlocks": "dungeon_cell_north_exit",
      "consumed_on_use": true
    },
    "rusty_sword": {
      "type": "passive",
      "effect": "flavor_dialogue"
    },
    "ancient_scroll": {
      "type": "quest_item",
      "use_on_npc": "ghost_knight",
      "triggers": "quest_complete",
      "consumed_on_use": true
    },
    "bone_amulet": {
      "type": "passive",
      "effect": "darkness_slow",
      "darkness_tick_interval": 7
    }
  }
}
```

- [ ] **Step 2: Write WORLD/entrance.md**

```markdown
# Castle Entrance

*The great iron gate stands behind you, fused shut by decades of rust and malice. Before you, the castle yawns open — cold, dark, and patient.*

---

## Description

You stand in the entrance hall of a vast and decaying castle. Stone columns flank a path northward into shadow. To your east, a narrow corridor leads to a lower passage. The air smells of wet stone and old fear. Above the gate, a carved inscription reads: **"What enters does not leave unchanged."**

---

## Items Here

- **Torch** — A guttering torch hangs in a rusted sconce near the gate.

---

## Exits

- **North** → The Great Hall
- **East** → Dungeon Cell

---

## Notes

*Gate (north of entrance, leading outside) requires `gate_unlocked` flag to be true.*
*When gate_unlocked is true, an additional exit appears: "Outside → ESCAPE".*
```

- [ ] **Step 3: Write WORLD/great_hall.md**

```markdown
# The Great Hall

*Centuries of darkness have made this room their home. The banners rotted away long ago. What remains are bones of grandeur — and something that refuses to leave.*

---

## Description

The Great Hall stretches vast and cold. A shattered chandelier hangs by a single chain, swaying without wind. Cracked stone floors are carpeted in ash. At the far end, where a throne once stood, a figure drifts — barely visible, a knight in dissolving armor, bound by chains of pale light.

---

## Items Here

*(none)*

---

## NPCs

- **Sir Aldric the Bound** — A ghost in ruined armor, chained to the center of the hall. His eyes are hollow lanterns.

---

## Exits

- **South** → Castle Entrance
- **West** → The Armory
- **Up** → The Watchtower
- **Down** → Dungeon Cell *(requires Iron Key)*
```

- [ ] **Step 4: Write WORLD/armory.md**

```markdown
# The Armory

*Once the pride of the castle garrison. Now a mausoleum of broken weapons and shattered purpose.*

---

## Description

Weapon racks line the walls, most holding only splinters or rust. The floor is littered with broken links of chainmail. A single weapon remains intact — a sword, eaten by rust but still dangerous in its desperation.

---

## Items Here

- **Rusty Sword** — Propped against the far wall, its blade pitted and brown but its edge still catching what little light exists.

---

## Exits

- **East** → The Great Hall
```

- [ ] **Step 5: Write WORLD/dungeon_cell.md**

```markdown
# Dungeon Cell

*Whoever was kept here was meant to be forgotten.*

---

## Description

A low, damp cell carved from the living rock beneath the castle. Iron rings are bolted to the walls. The floor is black with old moisture. A single barred window near the ceiling shows nothing but more stone. On the floor, half-buried in grime, something metal catches a sliver of light.

---

## Items Here

- **Iron Key** — Half-buried in the corner, its bow shaped like a serpent devouring its own tail.

---

## Exits

- **West** → Castle Entrance
- **North** → The Great Hall *(requires Iron Key — a locked iron door bars this passage)*
```

- [ ] **Step 6: Write WORLD/tower.md**

```markdown
# The Watchtower

*High above the curse, the tower breathes. Something was hidden here deliberately.*

---

## Description

A circular chamber at the top of a winding stair. Arrow slits look out over the dead landscape surrounding the castle. In the center of the room, a stone pedestal holds a single object, sealed and waiting. The air up here is different — thinner, older. As if time itself forgot to rot this place.

---

## Items Here

- **Ancient Scroll** — Resting on the stone pedestal. Sealed with black wax marked by a broken sigil. It hums faintly when touched.

---

## Exits

- **Down** → The Great Hall
```

- [ ] **Step 7: Commit**

```bash
cd d:/Code/dcastle
git add WORLD/
git commit -m "feat: add world registry and 5 room files"
git push
```

---

## Task 8: Create COMMANDS/ Files

**Files:**
- Create: `COMMANDS/move.md`
- Create: `COMMANDS/inspect.md`
- Create: `COMMANDS/take.md`
- Create: `COMMANDS/use.md`
- Create: `COMMANDS/talk.md`

- [ ] **Step 1: Write COMMANDS/move.md**

```markdown
# Command: MOVE

Triggered by: go, walk, head, move, run, north, south, east, west, up, down, n, s, e, w

---

## Resolution Algorithm

1. Parse direction from player input (north/south/east/west/up/down)
2. Read `STATE.json:player.current_room` → get current room ID
3. Read `WORLD/REGISTRY.json:rooms.<current_room>.exits`
4. Find exit matching the named direction

**If no exit in that direction:**
- Respond: "There is no path [direction]. [Describe what's there — wall, void, etc.]"
- Do NOT update STATE.json
- Do NOT increment turn_count

**If exit exists, check conditions:**

| Condition | Check | Blocked response |
|---|---|---|
| `requires_item` | item in `STATE.json:player.inventory` | "A [obstacle] blocks the way. [Hint about needed item]." |
| `requires_darkness_min` | `STATE.json:world.darkness_level >= value` | "There is nothing there." (room doesn't exist yet) |
| `requires_flag` | `STATE.json:world.flags.<flag> == true` | Context-appropriate block message |

**If item required and in inventory:**
- Consume item: set `items.<id>.location = "used"`, remove from inventory
- Allow movement

**If all conditions met:**
1. Set `STATE.json:player.current_room = <destination>`
2. Add destination to `STATE.json:world.rooms_visited` if not present
3. Increment `STATE.json:world.turn_count` by 1
4. Update `PLAYER.md`
5. Append to `LOGS/session.md`
6. Run darkness check
7. Narrate arrival in destination room (read WORLD/<destination>.md for description)

---

## Special: Escape

If current_room = "entrance" AND direction = "north" AND flags.gate_unlocked = true:
1. Set player.status = "escaped"
2. Increment turn_count
3. Append "THE WANDERER HAS ESCAPED" to LOGS/session.md
4. Narrate victory
```

- [ ] **Step 2: Write COMMANDS/inspect.md**

```markdown
# Command: INSPECT

Triggered by: look, examine, inspect, describe, what's here, where am i, survey, scan, l

---

## Resolution Algorithm

**INSPECT costs 0 turns.**

1. Read `STATE.json:player.current_room`
2. Read `WORLD/<current_room>.md` for base description
3. Build dynamic room report:

**Items present:**
- For each item in `STATE.json:items`: if `location == current_room` AND `taken == false`, list it
- If player carries torch: add one extra sensory detail to item descriptions

**NPCs present:**
- Check `STATE.json:npcs` for any npc with `location == current_room`
- If present: describe their current state (bound/freed)

**Exits available:**
- For each exit in REGISTRY.json for this room:
  - Only list exits whose conditions are currently met
  - Do NOT reveal exits that are darkness-gated and threshold not met

**Format:**
```
[Room Name]

[2-4 sentence atmospheric description]

Items: [list or "Nothing of note."]
NPCs: [list or none]
Exits: [list of currently available directions]
```

4. Append INSPECT entry to LOGS/session.md
5. Narrate

---

## Note on Torch

If `"torch"` is in `STATE.json:player.inventory`:
- Expand descriptions with one additional detail per room visit
- Example: "By torchlight, you notice scratch marks behind the weapon rack..."
```

- [ ] **Step 3: Write COMMANDS/take.md**

```markdown
# Command: TAKE

Triggered by: take, grab, pick up, get, collect, retrieve

---

## Resolution Algorithm

1. Parse item name from player input (fuzzy match to known item names)
2. Check `STATE.json:items` for an item whose:
   - name or id loosely matches the player's input
   - `location == STATE.json:player.current_room`
   - `taken == false`

**If no matching item in current room:**
- Respond: "There is no [item name] here."
- Do NOT update STATE.json
- Do NOT increment turn_count

**If item found:**
1. Set `STATE.json:items.<id>.location = "inventory"`
2. Set `STATE.json:items.<id>.taken = true`
3. Add `<id>` to `STATE.json:player.inventory`
4. Increment `STATE.json:world.turn_count` by 1
5. Update `PLAYER.md`
6. Update `INVENTORY/items.md`
7. Update `WORLD/<current_room>.md` — remove item from "Items Here" section
8. Append to `LOGS/session.md`
9. Run darkness check
10. Narrate pickup with atmospheric flavor

---

## Item Pickup Flavor

- torch: "You take the torch. Its warmth is faint comfort in the castle's chill."
- iron_key: "The key is cold and heavy. Something about its weight feels like obligation."
- rusty_sword: "You take the sword. The rust flakes onto your hand. Still, it has an edge."
- ancient_scroll: "The moment your fingers close around it, the hum intensifies. Something is sealed inside."
- bone_amulet: "It's unpleasant to touch. But immediately, the shadows seem slightly less aggressive."
```

- [ ] **Step 4: Write COMMANDS/use.md**

```markdown
# Command: USE

Triggered by: use, apply, insert, give, hand over, put, place, offer

---

## Resolution Algorithm

1. Parse item name from player input
2. Check `STATE.json:player.inventory` contains the item

**If item not in inventory:**
- Respond: "You don't have that."
- Do NOT update STATE.json
- Do NOT increment turn_count

**If item in inventory, check WORLD/REGISTRY.json:item_effects for effect type:**

### type: "key"
- Check if player is in the correct room for the key's use context
- Check if the relevant locked exit exists
- If yes: consume item, move player through exit (call MOVE resolution with condition bypassed)
- If no relevant use here: "There's nothing here to use that on."

### type: "quest_item" (ancient_scroll)
- Check if `target_npc` is in current room (`STATE.json:npcs.ghost_knight.location == current_room`)
- Check if NPC state == "bound"
- If yes: trigger quest_complete sequence (see SYSTEM.md §Sir Aldric)
- If NPC not in room: "There's no one here to give that to."
- If NPC already freed: "Sir Aldric regards the scroll, then you. 'It is already done,' he says."

### type: "passive"
- Respond: "The [item] doesn't seem to have a direct use. Perhaps it helps just by being carried."
- No state change, no turn consumed

---

## Quest Complete Sequence (triggered when scroll given to ghost_knight)

1. Remove ancient_scroll from inventory
2. Set `STATE.json:items.ancient_scroll.location = "used"`
3. Set `STATE.json:npcs.ghost_knight.state = "freed"`
4. Set `STATE.json:npcs.ghost_knight.quest_complete = true`
5. Set `STATE.json:world.darkness_level = 0`
6. Set `STATE.json:world.flags.gate_unlocked = true`
7. Increment turn_count
8. Update PLAYER.md, INVENTORY/items.md, NPCS/ghost_knight.md
9. Append dramatic entry to LOGS/session.md
10. Narrate: Sir Aldric's release, chains dissolving, castle shaking, light returning, gate grinding open
```

- [ ] **Step 5: Write COMMANDS/talk.md**

```markdown
# Command: TALK

Triggered by: talk, speak, ask, greet, address, call out to, say hello, say to

---

## Resolution Algorithm

1. Check `STATE.json:npcs` for any NPC with `location == STATE.json:player.current_room`

**If no NPC in room:**
- Respond: "There is no one here to speak with."
- Do NOT update STATE.json
- Do NOT increment turn_count

**If NPC present, read NPCS/<npc_id>.md for current state dialogue:**

For ghost_knight:
- If `spoken_to == false`: deliver first-encounter dialogue (warning + scroll hint)
  - Set `spoken_to = true`
- If `spoken_to == true` AND `state == "bound"`: deliver plea dialogue
- If `state == "freed"`: deliver gratitude dialogue

After delivering dialogue:
1. Set `STATE.json:npcs.<id>.spoken_to = true`
2. Increment `STATE.json:world.turn_count` by 1
3. Update `NPCS/ghost_knight.md` to reflect spoken_to state
4. Append to `LOGS/session.md`
5. Run darkness check

---

## Give-Item Branch

If player says "give [item] to [npc]":
- Delegate to USE command with item = parsed item, context = npc interaction
- This is equivalent to "use [item]" while NPC is present
```

- [ ] **Step 6: Commit**

```bash
cd d:/Code/dcastle
git add COMMANDS/
git commit -m "feat: add all 5 command logic files"
git push
```

---

## Task 9: Create NPC File

**Files:**
- Create: `NPCS/ghost_knight.md`

- [ ] **Step 1: Write NPCS/ghost_knight.md**

```markdown
# Sir Aldric the Bound

**NPC ID:** `ghost_knight`
**Location:** `great_hall` (permanent)
**Current State:** `bound`
**Spoken To:** `false`

---

## Description

Sir Aldric was the last knight of the castle before the curse fell. He died defending the gate — or so he believed. In truth, he was consumed from within, his soul shackled to the hall by the same darkness that claimed the castle. He appears as a figure in dissolving armor, chains of pale light wound through his form. His voice sounds like wind through ruined stone.

---

## State Machine

```
bound (spoken_to: false)
    → [player talks] → bound (spoken_to: true)
    
bound (spoken_to: true)  
    → [player gives ancient_scroll] → freed (quest_complete: true)
    
freed
    → no further transitions
```

---

## Dialogue

### First Encounter (spoken_to: false → true)

> *The figure turns. Two pale lights where eyes should be regard you without warmth or welcome.*
>
> "Another one comes to the castle. And like all the others, you will linger too long."
>
> *The chains pulse.*
>
> "I am Aldric. I was a knight. Now I am... this. The curse does not kill — it collects. It is patient. It waits for you to stop moving."
>
> *He looks at his bound hands.*
>
> "There is a way to break what was done to me. Somewhere in this castle, sealed away by the one who made me this — a scroll marked with a broken sigil. I have felt it. I have always felt it. Bring it to me. Before the darkness reaches ten."

---

### Repeat Encounter (spoken_to: true, state: bound)

> "You still have time. Find the scroll. It is sealed with black wax and a broken sigil. Somewhere above — the tower, I think. But hurry. Every moment you linger, the castle grows stronger."

---

### After Being Freed (state: freed)

> *Sir Aldric stands straight for the first time. The chains are gone. He is still translucent, still a ghost — but his eyes are bright now, and his voice carries warmth it lacked before.*
>
> "It is done. The curse that bound me is broken. You have done what a hundred others could not."
>
> *He gestures toward the entrance.*
>
> "The gate is open. Go. Leave this place. The darkness retreats but will not stay gone forever. This castle will hunger again — but not for you. Not today."
>
> *He begins to fade.*
>
> "Remember this, Wanderer: the castle is patient. But so can you be."

---

## Notes for Engine

- Sir Aldric never moves from great_hall
- He cannot be attacked (rusty_sword use on him: "The sword passes through him like smoke. He regards you with tired disappointment.")
- His spoken_to flag persists through darkness resets (he remembers)
- When quest_complete = true, update this file's "Current State" and "Spoken To" fields
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add NPCS/
git commit -m "feat: add Sir Aldric the Bound NPC file"
git push
```

---

## Task 10: Create Player Mirror Files

**Files:**
- Create: `PLAYER.md`
- Create: `INVENTORY/items.md`

- [ ] **Step 1: Write PLAYER.md**

```markdown
# The Wanderer

**Status:** Alive
**Health:** 100
**Current Location:** Castle Entrance

---

## Inventory

*(empty)*

---

## Journal

*You don't know how you came to be here. You know only that the gate behind you is sealed, and that somewhere in the darkness ahead, something waits to be found — or something waits to find you.*

---

> *This file is automatically maintained by the game engine.*
> *Do not edit manually — it will be overwritten on the next command.*
```

- [ ] **Step 2: Write INVENTORY/items.md**

```markdown
# Inventory

**The Wanderer carries:**

*(nothing)*

---

> *This file is automatically maintained by the game engine.*
> *Do not edit manually — it will be overwritten on the next command.*
```

- [ ] **Step 3: Commit**

```bash
cd d:/Code/dcastle
git add PLAYER.md INVENTORY/
git commit -m "feat: add player and inventory mirror files"
git push
```

---

## Task 11: Create LOGS/session.md

**Files:**
- Create: `LOGS/session.md`

- [ ] **Step 1: Write LOGS/session.md**

```markdown
# Session Log

> Append-only. Never edit existing entries. The engine writes one entry per command.

---

## Entry Format

```
[Turn N] [Command] → [Result]
[Narrative note]
```

---

## Log

```
[Turn 0] GAME START
The Wanderer enters the castle. The gate seals behind them.
Darkness Level: 1. The curse is patient.
```
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add LOGS/
git commit -m "feat: add session log"
git push
```

---

## Task 12: Create README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

```markdown
# dcastle

> *The castle is patient. Are you?*

A dark fantasy dungeon game where **the repository is the world**.

---

## How to Play

You don't run any code. You don't install anything.

**Open this repository in Cursor or Claude Code.**

Type your commands in the chat. The AI is your game engine.

---

## Getting Started

Say any of these to begin:

- `look` — examine where you are
- `go north` — move in a direction
- `take torch` — pick up an item
- `talk to the ghost` — speak with what haunts this place

---

## The Castle

You are **The Wanderer**, sealed inside a cursed castle. Somewhere within:

- A ghost knight has been chained here for centuries
- An ancient scroll holds the key to his release
- The darkness grows with every moment you linger

**Escape before the darkness reaches 10. Free Sir Aldric before it's too late.**

---

## Castle Map (partial)

```
         [Tower]
            |
           (down)
[Armory] ←→ [Great Hall] ←→ [???]
                |
           (locked door)
                |
[Entrance] ←→ [Dungeon Cell]
```

*The castle evolves. New passages appear as darkness grows.*

---

## Commands

| What you type | What happens |
|---|---|
| `look` / `inspect` | Examine your surroundings |
| `go [direction]` | Move (north, south, east, west, up, down) |
| `take [item]` | Pick up an item |
| `use [item]` | Use or give an item |
| `talk to [person]` | Speak with an NPC |
| `run tests` | Run the full test suite |
| `restart` | Reset the game |

---

## Rules

- All game state lives in `STATE.json` — there is no hidden state
- The AI engine reads `STATE.json`, applies rules from `SYSTEM.md`, and updates files
- The darkness level rises every 5 turns — it will kill you
- `LOGS/session.md` records everything that happens

---

## Files

| File | Purpose |
|---|---|
| `STATE.json` | Source of truth |
| `PLAYER.md` | Your current status |
| `INVENTORY/items.md` | What you're carrying |
| `WORLD/` | The castle rooms |
| `NPCS/` | Who haunts the castle |
| `LOGS/session.md` | What has happened |
| `TESTS/` | Game validation scenarios |
| `CLAUDE.md` | Engine rules |
| `SYSTEM.md` | Game mechanics |

---

*Built with Claude Code. The repository is the game.*
```

- [ ] **Step 2: Commit**

```bash
cd d:/Code/dcastle
git add README.md
git commit -m "feat: add player-facing README"
git push
```

---

## Task 13: GREEN Verification — Simulate All 18 Tests

**Goal:** Confirm all 18 tests pass by simulating them against the current file state.

- [ ] **Step 1: Simulate TEST-001 through TEST-005 (movement)**

For each test:
1. Read the Given state
2. Apply the When command per COMMANDS/move.md + SYSTEM.md
3. Check every Then assertion against STATE.json and file contents
4. Report PASS/FAIL

Expected: All 5 PASS (files now exist and rules are defined)

- [ ] **Step 2: Simulate TEST-006 through TEST-009 (inventory)**

Same process for inventory tests. Expected: All 4 PASS.

- [ ] **Step 3: Simulate TEST-010 through TEST-012 (NPC)**

Same process for NPC tests. Expected: All 3 PASS.

- [ ] **Step 4: Simulate TEST-013 through TEST-015 (consistency)**

Same process for state consistency tests. Expected: All 3 PASS.

- [ ] **Step 5: Simulate TEST-016 through TEST-018 (darkness)**

Same process for darkness tests. Expected: All 3 PASS.

- [ ] **Step 6: Fix any failures**

If any test fails:
- Identify which rule or file is missing/wrong
- Fix the specific file (do not rewrite unrelated content)
- Re-simulate the failing test until it passes

- [ ] **Step 7: Commit verification**

```bash
cd d:/Code/dcastle
git add .
git commit -m "test: GREEN - all 18 tests passing, game verified"
git push
```

---

## Task 14: Final Playability Verification

Self-critique loop — simulate 5 player commands step-by-step:

- [ ] **Command 1:** `look` — verify inspect returns room description, items, exits, no turn consumed
- [ ] **Command 2:** `take torch` — verify inventory updated, room updated, turn incremented
- [ ] **Command 3:** `go north` — verify move to great_hall, PLAYER.md updated, log appended
- [ ] **Command 4:** `talk to the ghost` — verify spoken_to = true, dialogue delivered, log appended
- [ ] **Command 5:** `go west` — verify move to armory, turn 3, darkness still 1

Confirm after each:
- STATE.json reflects expected values
- PLAYER.md mirrors STATE.json player section
- INVENTORY/items.md mirrors inventory
- LOGS/session.md has new entry

- [ ] **Final push**

```bash
cd d:/Code/dcastle
git add .
git commit -m "feat: complete dcastle v1.0 — AI-native dungeon castle game, all tests passing"
git push
```

---

## Self-Review

**Spec coverage check:**

| Spec requirement | Task that implements it |
|---|---|
| Movement between rooms | Task 5 (SYSTEM.md), Task 8 (COMMANDS/move.md) |
| Inspecting environment | Task 8 (COMMANDS/inspect.md) |
| Picking up items | Task 8 (COMMANDS/take.md) |
| Using items | Task 8 (COMMANDS/use.md) |
| NPC interaction | Task 9 (NPCS/ghost_knight.md), Task 8 (COMMANDS/talk.md) |
| Persistent state | Task 6 (STATE.json), all update steps |
| Narrative logging | Task 11 (LOGS/session.md), all append steps |
| All 18 tests | Task 3 (TESTS/), Task 13 (verification) |
| Self-evolving castle | CLAUDE.md §materialize_crypt, SYSTEM.md §Darkness |
| Agent file | Task 2 (.claude/agents/dcastle-engine.md) |
| GitHub publish | Task 1 (git init, gh repo create) |

All requirements covered. No placeholders. Types consistent across files (item IDs match between STATE.json, REGISTRY.json, COMMANDS/, and NPCS/).
