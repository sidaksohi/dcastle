# Test: State Consistency (TEST-013 to TEST-015)

These tests verify that STATE.json and all its mirror files stay in sync after every command.

---

## TEST-013: PLAYER.md always mirrors STATE.json player section

**Given:** Any game state where at least one command has been processed.

**When:** Read STATE.json and PLAYER.md immediately after any command completes.

**Then:**
- `PLAYER.md` Status field matches `STATE.json:player.status` ✓
- `PLAYER.md` Health field matches `STATE.json:player.health` ✓
- `PLAYER.md` Current Location field matches the display name of `STATE.json:player.current_room` ✓
  - "entrance" → "Castle Entrance"
  - "great_hall" → "The Great Hall"
  - "armory" → "The Armory"
  - "dungeon_cell" → "Dungeon Cell"
  - "tower" → "The Watchtower"
  - "crypt" → "The Crypt"
  - "void_corridor" → "The Void Corridor"
- `PLAYER.md` Inventory section lists exactly the same items as `STATE.json:player.inventory` ✓
- No field in PLAYER.md references state not present in STATE.json ✓

**Verification method:** AI reads both files and cross-checks all four fields. Any mismatch = FAIL.

---

## TEST-014: INVENTORY/items.md always mirrors STATE.json inventory

**Given:** Player has at least one item in inventory (e.g., after taking the torch).

**When:** Read STATE.json, INVENTORY/items.md, and the room the item was taken from.

**Then:**
- Every item ID in `STATE.json:player.inventory` appears by name in `INVENTORY/items.md` ✓
- No item appears in `INVENTORY/items.md` that is not in `STATE.json:player.inventory` ✓
- For each taken item: `STATE.json:items.<id>.location = "inventory"` and `taken = true` ✓
- The room file that originally contained the item (`WORLD/<original_room>.md`) no longer lists the item under "Items Here" ✓

**Example:** If inventory = ["torch", "iron_key"]:
- INVENTORY/items.md lists both Torch and Iron Key
- WORLD/entrance.md does not list Torch
- WORLD/dungeon_cell.md does not list Iron Key
- STATE.json: items.torch.location = "inventory", items.iron_key.location = "inventory"

**Verification method:** AI reads STATE.json, INVENTORY/items.md, and the relevant WORLD/ files and cross-checks. Any mismatch = FAIL.

---

## TEST-015: LOGS/session.md is strictly append-only

**Given:** `LOGS/session.md` has N existing entries.

**When:** Any command is processed (the Nth+1 command).

**Then:**
- `LOGS/session.md` has exactly N+1 entries ✓
- The first N entries are character-for-character identical to their state before the command ✓
- The new (N+1th) entry contains:
  - Turn number ✓
  - The command that was issued ✓
  - A brief result summary ✓
- No existing entry was modified, deleted, or reordered ✓

**Verification method:** AI reads LOGS/session.md before a command, stores the content, runs the command, reads the file again, and confirms: (a) old content unchanged at the top, (b) new entry appended at the bottom.

**Note:** This test also applies to failed/invalid commands — even those that don't change game state must still append an entry to the log.
