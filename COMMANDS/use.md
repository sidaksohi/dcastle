# Command: USE

**Triggers:** use, apply, give, hand over, offer, insert, put, place

---

## Resolution Algorithm

### Step 1 — Parse item and optional target
Extract item reference (and optional target NPC/object) from player input.
Resolve to canonical item ID using CLAUDE.md §Item Name Resolution.

### Step 2 — Check inventory
Confirm `<id>` is in `STATE.json:player.inventory`.

**If item NOT in inventory:**
- Respond: "You don't have that."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] USE {item} → not in inventory`
- Stop.

### Step 3 — Resolve effect type from WORLD/REGISTRY.json:item_effects

---

## Effect Type: passive

Applies to: `torch`, `rusty_sword`

- Respond: "The [item name] doesn't seem to have a direct use here. Perhaps simply carrying it helps."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] USE {item} → passive item, no direct use`

---

## Effect Type: key

Applies to: `iron_key`

1. Check if player is in `dungeon_cell`
2. Check if north exit exists and has `requires_item: "iron_key"`

**If in dungeon_cell:**
- Consume key: `items.iron_key.location = "used"`, remove from `player.inventory`
- Execute move to `great_hall` (follow COMMANDS/move.md Step 4 with condition bypassed)
- Narrate: "The iron key turns with a grinding shriek. The door grinds open. You step through."

**If NOT in dungeon_cell:**
- Respond: "There's no lock here that the key fits."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] USE iron_key → no applicable lock in {room}`

---

## Effect Type: quest_item

Applies to: `ancient_scroll`

1. Check if `ghost_knight` is in current room (`npcs.ghost_knight.location == current_room`)
2. Check if `npcs.ghost_knight.state == "bound"`

**Condition: NPC in room AND state is "bound":**

### Quest Complete Sequence

1. `STATE.json:items.ancient_scroll.location = "used"`
2. Remove `"ancient_scroll"` from `STATE.json:player.inventory`
3. `STATE.json:npcs.ghost_knight.state = "freed"`
4. `STATE.json:npcs.ghost_knight.quest_complete = true`
5. `STATE.json:world.darkness_level = 0`
6. `STATE.json:world.flags.gate_unlocked = true`
7. `STATE.json:world.turn_count += 1`
8. Write `STATE.json`
9. Write `PLAYER.md`
10. Write `INVENTORY/items.md` (remove scroll)
11. Update `NPCS/ghost_knight.md`:
    - Change `**Current State:** bound` → `**Current State:** freed`
    - Change `**Quest Complete:** false` → `**Quest Complete:** true`
12. Update `WORLD/great_hall.md` NPC section to reflect freed state
13. Append to LOGS: `[Turn {N}] QUEST COMPLETE → Sir Aldric freed. Darkness reset. Gate unlocked.`
14. Run darkness check (darkness is 0, nothing triggers)
15. Narrate: Sir Aldric's dramatic release (use the "After Being Freed" dialogue from NPCS/ghost_knight.md)

**Condition: NPC NOT in room:**
- Respond: "There is no one here to give that to."
- No state change. No turn consumed.

**Condition: NPC in room, state is "freed":**
- Respond: "Sir Aldric regards the scroll with gentle eyes. 'It is already done,' he says."
- No state change. No turn consumed.

---

## Give-Item Shorthand

"give [item] to [target]" is equivalent to "use [item]" — delegate to this command.
The target name (e.g., "knight", "ghost", "Aldric") is resolved to the NPC and used for context in error messages.
