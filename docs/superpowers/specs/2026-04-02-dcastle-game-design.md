# dcastle — AI-Native Repository Game Design Spec

**Date:** 2026-04-02
**Theme:** Dark Fantasy Dungeon Castle (inspired by Dracula's self-evolving castle)
**Engine:** AI (Claude) reads files → applies rules → updates files → narrates result
**Tests:** Structured Markdown scenarios (Given/When/Then), AI simulates them

---

## 1. Core Philosophy

The repository IS the game. No code runs. The player types natural language to the AI.
The AI is a deterministic game engine:
1. Read STATE.json and relevant files
2. Apply rules from SYSTEM.md and COMMANDS/
3. Update STATE.json, PLAYER.md, WORLD/ files, INVENTORY/items.md
4. Append to LOGS/session.md
5. Narrate what happened

STATE.json is the single source of truth. If it is not in the repo, it does not exist.

---

## 2. File Structure

```
dcastle/
├── README.md                        # Player instructions
├── CLAUDE.md                        # AI engine contract
├── SYSTEM.md                        # Game engine rules & mechanics
├── STATE.json                       # Single source of truth
├── PLAYER.md                        # Human-readable player state mirror
│
├── .claude/
│   └── agents/
│       └── dcastle-engine.md        # Claude Code agent definition
│
├── WORLD/
│   ├── REGISTRY.json               # Room graph, exits, conditions, latent rooms
│   ├── entrance.md
│   ├── great_hall.md
│   ├── armory.md
│   ├── dungeon_cell.md
│   └── tower.md
│
├── INVENTORY/
│   └── items.md                    # Human-readable inventory mirror
│
├── NPCS/
│   └── ghost_knight.md             # Sir Aldric the Bound
│
├── COMMANDS/
│   ├── move.md
│   ├── inspect.md
│   ├── take.md
│   ├── use.md
│   └── talk.md
│
├── LOGS/
│   └── session.md                  # Append-only narrative log
│
└── TESTS/
    ├── README.md
    ├── test_movement.md
    ├── test_inventory.md
    ├── test_npc.md
    ├── test_state_consistency.md
    └── test_darkness.md
```

---

## 3. STATE.json Schema

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
    "torch": { "location": "entrance", "taken": false },
    "iron_key": { "location": "dungeon_cell", "taken": false },
    "rusty_sword": { "location": "armory", "taken": false },
    "ancient_scroll": { "location": "tower", "taken": false }
  }
}
```

---

## 4. Castle Map (MVP)

```
      [Tower]
         |
         | (down)
[Armory]←→[Great Hall]←→[Crypt*]
              |
           (down, req: iron_key)
              |
[Entrance]←→[Dungeon Cell]
```

*Crypt materializes at darkness_level >= 5

### Room Details

| Room | Key Item | NPC | Exits |
|---|---|---|---|
| entrance | torch | — | north→great_hall, east→dungeon_cell |
| great_hall | — | ghost_knight | south→entrance, west→armory, up→tower, down→dungeon_cell (req: iron_key), east→crypt (req: darkness>=5) |
| armory | rusty_sword | — | east→great_hall |
| dungeon_cell | iron_key | — | west→entrance, north→great_hall (req: iron_key) |
| tower | ancient_scroll | — | down→great_hall |
| crypt* | bone_amulet* | — | west→great_hall |

---

## 5. Darkness Mechanics (Self-Evolution)

- Starts at 1
- Increases by 1 every 5 turns
- `darkness_level >= 5`: crypt materializes (WORLD/crypt.md written, great_hall exits updated)
- `darkness_level >= 8`: void_corridor materializes off the crypt
- `darkness_level = 10`: player.status = "consumed", game over

The castle evolves as the player lingers. Urgency is mechanical, not cosmetic.

---

## 6. NPC: Sir Aldric the Bound

- Location: great_hall
- State machine: `bound` → (give ancient_scroll) → `freed`
- When `spoken_to=false`: delivers cryptic warning about the curse
- When `spoken_to=true` and player gives ancient_scroll:
  - ghost_knight.state = "freed", quest_complete = true
  - darkness_level resets to 0
  - flags.gate_unlocked = true
  - Darkness evolution pauses (won path begins)

---

## 7. Win Condition

1. Collect iron_key (dungeon_cell)
2. Collect ancient_scroll (tower)
3. Give ancient_scroll to Sir Aldric → darkness resets, gate unlocks
4. Return to entrance, use iron_key → ESCAPE → player.status = "escaped" → WIN

---

## 8. Command Dispatch

The AI maps natural language to one of five canonical commands:

| Command | Triggers | Effect |
|---|---|---|
| MOVE | go, walk, head, move, north/south/east/west/up/down | Update current_room if exit valid |
| INSPECT | look, examine, inspect, describe, what's here | Narrate room from room file |
| TAKE | take, grab, pick up, get | Move item from room to inventory |
| USE | use, apply, insert, give | Apply item effect |
| TALK | talk, speak, ask, greet | Trigger NPC dialogue |

Unknown commands: AI responds "The castle doesn't understand." No state change.

---

## 9. Consistency Rules

After EVERY command:
1. STATE.json is updated first
2. PLAYER.md is rewritten to mirror player section of STATE.json
3. INVENTORY/items.md is rewritten to mirror inventory
4. If room changed: WORLD/<room>.md present item list updated
5. LOGS/session.md gets one new entry appended (timestamp + turn + action + result)
6. Darkness check: if turn_count % 5 == 0 and turn_count > 0, darkness_level++

---

## 10. Tests Summary

18 structured scenarios across 5 test files covering:
- Valid and invalid movement
- Darkness-gated room materialization
- Item pickup (present/absent)
- Item use (owned/not owned)
- NPC interaction state machine
- Quest completion and win condition
- STATE.json ↔ markdown file consistency
- Append-only log verification
- Darkness escalation and game over
