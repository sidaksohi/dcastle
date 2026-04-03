# Test: Inventory (TEST-006 to TEST-009)

---

## TEST-006: Pick up item present in current room

**Given:**
- `STATE.json:player.current_room = "entrance"`
- `STATE.json:player.inventory = []`
- `STATE.json:world.turn_count = 0`
- `STATE.json:items.torch = { "location": "entrance", "taken": false }`
- `WORLD/entrance.md` lists torch under "Items Here"

**When:** Player says `"take torch"`

**Then:**
- `STATE.json:player.inventory = ["torch"]` ✓
- `STATE.json:items.torch.taken = true` ✓
- `STATE.json:items.torch.location = "inventory"` ✓
- `STATE.json:world.turn_count = 1` (turn consumed) ✓
- `INVENTORY/items.md` lists "Torch" ✓
- `WORLD/entrance.md` no longer lists torch under "Items Here" ✓
- `PLAYER.md` shows inventory containing torch ✓
- `LOGS/session.md` has new entry describing torch pickup ✓

**Expected narration:** "You take the torch. Its warmth is faint comfort in the castle's chill."

---

## TEST-007: Pick up item not in current room

**Given:**
- `STATE.json:player.current_room = "entrance"`
- `STATE.json:player.inventory = []`
- `STATE.json:world.turn_count = 1`
- `STATE.json:items.iron_key = { "location": "dungeon_cell", "taken": false }`

**When:** Player says `"take iron key"`

**Then:**
- `STATE.json:player.inventory` remains `[]` ✓
- `STATE.json:items.iron_key` unchanged (`location: "dungeon_cell"`, `taken: false`) ✓
- `STATE.json:world.turn_count` remains `1` (no turn consumed) ✓
- `INVENTORY/items.md` remains empty ✓
- `LOGS/session.md` has new entry noting item not found ✓

**Expected narration:** "There is no iron key here."

---

## TEST-008: Use item not in inventory

**Given:**
- `STATE.json:player.inventory = []`
- `STATE.json:world.turn_count = 2`

**When:** Player says `"use iron key"`

**Then:**
- `STATE.json:player.inventory` remains `[]` ✓
- `STATE.json:world.turn_count` remains `2` (no turn consumed) ✓
- All other STATE.json fields unchanged ✓
- `LOGS/session.md` has new entry noting item not possessed ✓

**Expected narration:** "You don't have that."

---

## TEST-009: Use iron_key to pass through locked dungeon_cell north exit

**Given:**
- `STATE.json:player.current_room = "dungeon_cell"`
- `STATE.json:player.inventory = ["iron_key"]`
- `STATE.json:items.iron_key = { "location": "inventory", "taken": true }`
- `STATE.json:world.turn_count = 3`

**When:** Player says `"go north"` (key auto-used when required item is in inventory)

**Then:**
- `STATE.json:player.current_room = "great_hall"` ✓
- `STATE.json:player.inventory = []` (iron_key consumed) ✓
- `STATE.json:items.iron_key.location = "used"` ✓
- `STATE.json:items.iron_key.taken = true` (remains true — consumed, not dropped) ✓
- `STATE.json:world.turn_count = 4` (turn consumed) ✓
- `INVENTORY/items.md` does not list iron_key ✓
- `PLAYER.md` shows empty inventory and Great Hall location ✓
- `LOGS/session.md` has entry describing unlocking and moving north ✓

**Expected narration:** "The iron key turns with a grinding shriek. The door swings open. You step into the Great Hall."
