# SYSTEM.md — dcastle Game Mechanics

This document defines every mechanical rule. The AI engine consults this before resolving any command.

---

## The World

**dcastle** is a cursed castle. Darkness is both atmosphere and timer.

The player must find Sir Aldric the Bound, discover the ancient scroll, and free him before the darkness level reaches 10 and the castle consumes them.

---

## Initial State Values (for restart)

```json
{
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
    "torch":         { "location": "entrance",     "taken": false },
    "iron_key":      { "location": "dungeon_cell", "taken": false },
    "rusty_sword":   { "location": "armory",       "taken": false },
    "ancient_scroll":{ "location": "tower",        "taken": false },
    "bone_amulet":   { "location": "crypt",        "taken": false }
  }
}
```

---

## Rooms

The active world graph lives in `WORLD/REGISTRY.json`. Each room has:
- `id` — file ID (matches `WORLD/<id>.md`)
- `name` — display name shown to player
- `exits` — direction → destination, with optional conditions
- `items_initial` — items present at game start
- `npc` — NPC ID if one is present (null otherwise)
- `latent` — true if room starts unformed and materializes during play

### Exit Conditions

| Condition key | How it's checked |
|---------------|-----------------|
| `requires_item` | Item ID must be in `STATE.json:player.inventory` |
| `requires_darkness_min` | `STATE.json:world.darkness_level >= value` |
| `requires_flag` | `STATE.json:world.flags.<flag_name> == true` |

If a condition is not met, movement is blocked. No turn consumed. AI explains the obstacle thematically (no mechanical jargon).

If `requires_item` IS met: the item is **consumed** on use (location = "used", removed from inventory).

---

## Movement Rules

1. Parse direction from player input
2. Look up exit in `WORLD/REGISTRY.json:rooms.<current_room>.exits`
3. If no exit in that direction → blocked ("no path that way")
4. If exit exists, evaluate conditions top-to-bottom:
   - Any failing condition → blocked with thematic explanation
   - All conditions passing → allow move
5. If `requires_item` triggered and passing: consume the item
6. Update `STATE.json:player.current_room`
7. Add destination to `STATE.json:world.rooms_visited` if not present
8. **AUTO-PICKUP:** All items in the destination room are automatically added to inventory
   - For each item: `location = "inventory"`, `taken = true`, added to `player.inventory`
   - `WORLD/<destination>.md` Items Here section updated to `*(none)*`
   - `INVENTORY/items.md` and `PLAYER.md` updated
9. Increment `turn_count`
10. Run darkness check
11. Narrate arrival using `WORLD/<destination>.md`, weaving in any picked-up items naturally

### Special: Escape from Entrance

When moving south from entrance AND `flags.gate_unlocked == true`:
- Set `player.status = "escaped"`
- This is the win condition

---

## INSPECT Rules

- Free action — does NOT consume a turn
- Reads `WORLD/<current_room>.md` for base description
- Lists items: all items where `STATE.json:items.<id>.location == current_room` AND `taken == false`
- Lists NPCs: all npcs where `STATE.json:npcs.<id>.location == current_room`
- Lists exits: all exits in REGISTRY.json for the room whose conditions are currently met
  - Hidden exits (conditions not met) are NOT revealed
- If player carries `torch`: add one extra atmospheric detail to the description
- Appends to log (but with a `[free]` tag — no turn number increment)

---

## TAKE Rules

1. Player names an item
2. Engine resolves item ID from player's words (see CLAUDE.md §Item Name Resolution)
3. Check: `STATE.json:items.<id>.location == current_room` AND `taken == false`
   - If not found: "There is no [item] here." — no state change, no turn
4. If found:
   - `items.<id>.location = "inventory"`, `items.<id>.taken = true`
   - `player.inventory.push(item_id)`
   - Increment `turn_count`
   - Run darkness check
   - Update PLAYER.md, INVENTORY/items.md, WORLD/<room>.md (remove item)

---

## USE Rules

1. Player names an item (and optionally a target)
2. Check `player.inventory` contains the item
   - If not: "You don't have that." — no state change, no turn
3. Look up `item_effects` in `WORLD/REGISTRY.json`

| Effect type | Behavior |
|-------------|----------|
| `passive` | "The [item] doesn't have a direct use here. Perhaps simply carrying it helps." — no state change, no turn |
| `key` | Attempt to use on relevant locked exit (if in correct room). Consume item, trigger move. |
| `quest_item` | If target NPC in room and NPC state allows: trigger quest sequence. Else explain. |

### Key Use

`iron_key` can be used from `dungeon_cell` going north, OR by simply attempting to go north with the key in inventory.

### Quest Item Use (ancient_scroll)

Conditions:
1. Player is in `great_hall`
2. `ghost_knight.state == "bound"`
3. `ancient_scroll` in inventory

If all conditions met → Quest Complete Sequence:
1. `items.ancient_scroll.location = "used"`, remove from inventory
2. `npcs.ghost_knight.state = "freed"`
3. `npcs.ghost_knight.quest_complete = true`
4. `world.darkness_level = 0`
5. `world.flags.gate_unlocked = true`
6. Increment `turn_count`
7. Update all mirrors, append dramatic log entry
8. Narrate Sir Aldric's release

If NPC not in room: "There is no one here to give that to."
If NPC already freed: "Sir Aldric regards the scroll with gentle eyes. 'It is already done,' he says."

---

## TALK Rules

1. Check for NPC in current room: any `STATE.json:npcs.<id>.location == current_room`
   - If none: "There is no one here to speak with." — no state change, no turn
2. Load `NPCS/<npc_id>.md` for current dialogue state
3. Set `npcs.<id>.spoken_to = true`
4. Increment `turn_count`, run darkness check
5. Deliver dialogue based on NPC state (see NPCS/ghost_knight.md)

### Give-Item Branch

"give [item] to [NPC]" → delegate to USE with the item and NPC as context.

---

## Sir Aldric — State Machine

```
[bound / spoken_to: false]
    → TALK → [bound / spoken_to: true]   (first encounter dialogue)

[bound / spoken_to: true]
    → TALK → [bound / spoken_to: true]   (repeat plea)
    → USE ancient_scroll → [freed / quest_complete: true]

[freed / quest_complete: true]
    → TALK → [freed]                     (gratitude dialogue, no further change)
```

Sir Aldric never moves. He cannot be harmed. He persists across darkness resets.

---

## Items

| ID | Name | Location (start) | Effect |
|----|------|-------------------|--------|
| `torch` | Torch | entrance | Passive: enhances INSPECT descriptions |
| `iron_key` | Iron Key | dungeon_cell | Key: unlocks dungeon_cell north exit (consumed) |
| `rusty_sword` | Rusty Sword | armory | Passive: flavor dialogue changes with Sir Aldric |
| `ancient_scroll` | Ancient Scroll | tower | Quest: frees Sir Aldric when given to him |
| `bone_amulet` | Bone Amulet | crypt | Passive: darkness tick slows from 5→7 turns |

### Bone Amulet Effect

When `bone_amulet` is in `player.inventory`, replace the darkness tick formula:
- Default: `if turn_count % 5 == 0` → darkness +1
- With amulet: `if turn_count % 7 == 0` → darkness +1

This must be checked each time the darkness formula runs.

---

## Darkness

| Level | Castle state |
|-------|-------------|
| 1 | The castle is cold and still |
| 2–3 | Shadows move at the edge of vision |
| 4 | The walls seem closer. Something watches. |
| 5 | **Crypt materializes** — eastern passage opens in Great Hall |
| 6–7 | Descriptions grow oppressive. The air thickens. |
| 8 | **Void Corridor materializes** — northern passage opens in Crypt |
| 9 | Final warning: "THE DARKNESS WILL CLAIM YOU SOON." |
| 10 | **GAME OVER** — player.status = "consumed" |

### Darkness Tick Formula

```
tick_interval = 7 if "bone_amulet" in player.inventory else 5

if turn_count > 0 AND turn_count % tick_interval == 0:
    darkness_level += 1
```

---

## Turn Economics

| Action | Turn cost |
|--------|-----------|
| MOVE (valid, conditions met) | +1 |
| TAKE (item found in room) | +1 |
| USE (item in inventory, effect triggers) | +1 |
| TALK (NPC present) | +1 |
| INSPECT | 0 (free) |
| Invalid / failed action | 0 |
| Unknown command | 0 |

---

## Win / Lose Summary

**Win path:**
1. Take `iron_key` from dungeon_cell
2. Take `ancient_scroll` from tower
3. Speak with Sir Aldric (spoken_to = true)
4. Give scroll to Sir Aldric → darkness resets to 0, gate unlocks
5. Return to entrance, go south → escaped (gate is now open behind you)

**Lose condition:**
- darkness_level reaches 10 → consumed

**Optimal run:** ~9 turns minimum (go east, take key, go north, go up, take scroll, go down, give scroll, go south, go south to escape). Darkness would be 1 still on a perfect run.
