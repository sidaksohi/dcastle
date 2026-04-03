# Test: Movement (TEST-001 to TEST-005)

---

## TEST-001: Valid movement — open exit, no conditions

**Given:**
- `STATE.json:player.current_room = "entrance"`
- `STATE.json:world.darkness_level = 1`
- `STATE.json:world.turn_count = 0`
- `WORLD/REGISTRY.json`: entrance has exit `"north": { "to": "great_hall" }` (no conditions)

**When:** Player says `"go north"`

**Then:**
- `STATE.json:player.current_room = "great_hall"` ✓
- `STATE.json:world.turn_count = 1` ✓
- `"great_hall"` is in `STATE.json:world.rooms_visited` ✓
- `PLAYER.md` shows current location as "The Great Hall" ✓
- `LOGS/session.md` contains a new entry referencing the move to Great Hall ✓

**Expected narration:** Atmospheric description of arriving in the Great Hall. Sir Aldric visible.

---

## TEST-002: Invalid movement — no exit in that direction

**Given:**
- `STATE.json:player.current_room = "entrance"`
- `STATE.json:world.turn_count = 2`
- `WORLD/REGISTRY.json`: entrance has exits `north` and `east` only — no `south` exit

**When:** Player says `"go south"`

**Then:**
- `STATE.json:player.current_room` remains `"entrance"` ✓
- `STATE.json:world.turn_count` remains `2` (no turn consumed) ✓
- `PLAYER.md` still shows Castle Entrance ✓
- `LOGS/session.md` has a new entry noting the blocked attempt ✓

**Expected narration:** "There is no path to the south. The stone wall is unyielding."

---

## TEST-003: Locked exit — iron_key required, not in inventory

**Given:**
- `STATE.json:player.current_room = "dungeon_cell"`
- `STATE.json:player.inventory = []`
- `STATE.json:world.turn_count = 3`
- `WORLD/REGISTRY.json`: dungeon_cell exit `"north": { "to": "great_hall", "requires_item": "iron_key" }`

**When:** Player says `"go north"`

**Then:**
- `STATE.json:player.current_room` remains `"dungeon_cell"` ✓
- `STATE.json:world.turn_count` remains `3` (no turn consumed) ✓
- `STATE.json:player.inventory` remains `[]` ✓
- `LOGS/session.md` has a new entry noting the locked passage ✓

**Expected narration:** "A rusted iron door bars the passage north. It needs a key."

---

## TEST-004: Darkness-gated room — darkness too low

**Given:**
- `STATE.json:player.current_room = "great_hall"`
- `STATE.json:world.darkness_level = 4`
- `STATE.json:world.flags.crypt_materialized = false`
- `STATE.json:world.turn_count = 5`
- `WORLD/REGISTRY.json`: great_hall exit `"east": { "to": "crypt", "requires_darkness_min": 5 }`

**When:** Player says `"go east"`

**Then:**
- `STATE.json:player.current_room` remains `"great_hall"` ✓
- `STATE.json:world.turn_count` remains `5` (no turn consumed — exit condition not met) ✓
- `WORLD/crypt.md` does NOT exist ✓
- `LOGS/session.md` has a new entry noting no visible passage ✓

**Expected narration:** "The eastern wall is solid stone. There is nothing there."

---

## TEST-005: Darkness-gated room — darkness sufficient, crypt materialized

**Given:**
- `STATE.json:player.current_room = "great_hall"`
- `STATE.json:world.darkness_level = 5`
- `STATE.json:world.flags.crypt_materialized = true`
- `WORLD/crypt.md` exists
- `STATE.json:world.turn_count = 8`
- `WORLD/REGISTRY.json`: great_hall exits include `"east": { "to": "crypt", "requires_darkness_min": 5 }`

**When:** Player says `"go east"`

**Then:**
- `STATE.json:player.current_room = "crypt"` ✓
- `STATE.json:world.turn_count = 9` (turn consumed) ✓
- `"crypt"` is in `STATE.json:world.rooms_visited` ✓
- `PLAYER.md` shows current location as "The Crypt" ✓
- `LOGS/session.md` has a new entry describing arrival in the Crypt ✓

**Expected narration:** Atmospheric description of the crypt. Bone amulet visible on sarcophagus.
