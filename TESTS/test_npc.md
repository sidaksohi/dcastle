# Test: NPC Interaction (TEST-010 to TEST-012)

---

## TEST-010: Talk to NPC in same room — first encounter

**Given:**
- `STATE.json:player.current_room = "great_hall"`
- `STATE.json:world.turn_count = 2`
- `STATE.json:npcs.ghost_knight.location = "great_hall"`
- `STATE.json:npcs.ghost_knight.state = "bound"`
- `STATE.json:npcs.ghost_knight.spoken_to = false`

**When:** Player says `"talk to the ghost"` or `"speak with the knight"`

**Then:**
- `STATE.json:npcs.ghost_knight.spoken_to = true` ✓
- `STATE.json:npcs.ghost_knight.state` remains `"bound"` ✓
- `STATE.json:world.turn_count = 3` (turn consumed) ✓
- `NPCS/ghost_knight.md` field `Spoken To:` updated to `true` ✓
- `LOGS/session.md` has entry with dialogue exchange logged ✓

**Expected narration:** Sir Aldric's first-encounter dialogue. He warns about the curse. He hints at the scroll in the tower. His eyes are hollow lanterns.

---

## TEST-011: Talk command when no NPC in current room

**Given:**
- `STATE.json:player.current_room = "entrance"`
- `STATE.json:world.turn_count = 1`
- `STATE.json:npcs.ghost_knight.location = "great_hall"` (not in entrance)

**When:** Player says `"talk to the ghost"`

**Then:**
- `STATE.json` completely unchanged ✓
- `STATE.json:world.turn_count` remains `1` (no turn consumed) ✓
- `STATE.json:npcs.ghost_knight.spoken_to` unchanged ✓
- `LOGS/session.md` has entry noting no one present ✓

**Expected narration:** "There is no one here to speak with."

---

## TEST-012: Give ancient_scroll to Sir Aldric — quest completion

**Given:**
- `STATE.json:player.current_room = "great_hall"`
- `STATE.json:player.inventory = ["ancient_scroll"]`
- `STATE.json:items.ancient_scroll = { "location": "inventory", "taken": true }`
- `STATE.json:npcs.ghost_knight.state = "bound"`
- `STATE.json:npcs.ghost_knight.spoken_to = true`
- `STATE.json:npcs.ghost_knight.quest_complete = false`
- `STATE.json:world.darkness_level = 6`
- `STATE.json:world.flags.gate_unlocked = false`
- `STATE.json:world.turn_count = 12`

**When:** Player says `"give scroll to the knight"` or `"use ancient scroll"`

**Then:**
- `STATE.json:npcs.ghost_knight.state = "freed"` ✓
- `STATE.json:npcs.ghost_knight.quest_complete = true` ✓
- `STATE.json:player.inventory = []` (scroll consumed) ✓
- `STATE.json:items.ancient_scroll.location = "used"` ✓
- `STATE.json:world.darkness_level = 0` (curse shattered, darkness reset) ✓
- `STATE.json:world.flags.gate_unlocked = true` ✓
- `STATE.json:world.turn_count = 13` (turn consumed) ✓
- `INVENTORY/items.md` does not list ancient_scroll ✓
- `NPCS/ghost_knight.md` Current State field shows "freed" ✓
- `NPCS/ghost_knight.md` Quest Complete field shows "true" ✓
- `LOGS/session.md` has dramatic quest-completion entry ✓

**Expected narration:** Sir Aldric's chains dissolve. Light floods the hall. The darkness retreats. The gate grinds open in the distance. He delivers his final words before fading.
