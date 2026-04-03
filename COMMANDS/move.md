# Command: MOVE

**Triggers:** go, walk, head, move, run, north, south, east, west, up, down, n, s, e, w

---

## Resolution Algorithm

### Step 1 — Parse direction
Extract direction from player input. Normalize aliases:
- n → north, s → south, e → east, w → west, u → up, d → down

### Step 2 — Look up exit
Read `WORLD/REGISTRY.json:rooms.<current_room>.exits`
Find entry matching the named direction.

**If no exit in that direction:**
- Respond: "There is no path [direction]. [Thematic description of what's there — wall, void, locked gate]."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] MOVE {direction} → blocked: no exit`

### Step 3 — Evaluate conditions (if exit exists)

Process conditions in this order:

| Condition | Pass? | Action |
|-----------|-------|--------|
| `requires_item` | Item in inventory | Consume item on move (set location="used", remove from inventory) |
| `requires_item` | Item NOT in inventory | Blocked: thematic explanation of obstacle |
| `requires_darkness_min` | darkness_level >= value | Allow |
| `requires_darkness_min` | darkness_level < value | Blocked: "There is nothing there." (room doesn't exist yet) |
| `requires_flag` | Flag is true | Allow |
| `requires_flag` | Flag is false | Blocked: context-appropriate explanation |

**If any condition blocks:**
- No state change. No turn consumed.
- Append to LOGS: `[blocked] MOVE {direction} → condition not met`

### Step 4 — Execute move (all conditions pass)

1. `STATE.json:player.current_room = <destination_id>`
2. Add destination to `STATE.json:world.rooms_visited` if not already present
3. If `requires_item` was in the exit: set `items.<id>.location = "used"`, remove `<id>` from `player.inventory`
4. **AUTO-PICKUP:** For every item where `location == destination_id` AND `taken == false`:
   - Set `items.<id>.location = "inventory"`, `items.<id>.taken = true`
   - Add `<id>` to `player.inventory`
   - Remove item bullet from `WORLD/<destination>.md` Items Here section
5. `STATE.json:world.turn_count += 1`
6. Write `STATE.json`
7. Write `PLAYER.md` (update location and inventory)
8. Write `INVENTORY/items.md` (add any auto-picked items, remove any consumed key)
9. Write `WORLD/<destination>.md` (remove auto-picked items from Items Here)
10. Append to LOGS: `[Turn {N}] MOVE {direction} → arrived in {destination_name}` + list any auto-picked items
11. Run darkness check (see CLAUDE.md §Darkness Check)
12. Narrate arrival using `WORLD/<destination>.md` description, mentioning picked-up items naturally

### Step 5 — Special: Escape from entrance

If origin = "entrance" AND direction = "south" AND `flags.gate_unlocked == true`:
1. `player.status = "escaped"`
2. Append to LOGS: `[Turn {N}] ESCAPE → THE WANDERER HAS ESCAPED THE CASTLE.`
3. Narrate victory — the gate swings open, dawn light, freedom
4. No further commands accepted (terminal state)

---

## Response Tone

- Blocked by wall: "The stone is unyielding. There is no passage [direction]."
- Blocked by locked door: "A [door/gate/iron barrier] prevents passage. [Hint at what's needed]."
- Blocked by darkness gate: "The [wall/stone] is solid. There is nothing there." (Do NOT hint that a room will appear.)
- Arrival: 2-4 sentences describing the destination room atmosphere.
