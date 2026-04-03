# Test: Darkness and Castle Evolution (TEST-016 to TEST-018)

---

## TEST-016: Darkness level increments every 5 valid turns

**Given:**
- `STATE.json:world.darkness_level = 1`
- `STATE.json:world.turn_count = 4`

**When:** Player takes any valid action that consumes a turn (e.g., `"go north"`) — turn_count becomes 5.

**Then:**
- `STATE.json:world.turn_count = 5` ✓
- `STATE.json:world.darkness_level = 2` ✓
  (Because 5 % 5 == 0 and turn_count > 0 → darkness_level += 1)
- `PLAYER.md` still reflects the player's current state accurately ✓
- `LOGS/session.md` has an entry noting the darkness deepening ✓

**Expected narration:** Woven into the turn narration — "The shadows lengthen. The darkness deepens." or similar.

**Edge case:** If turn_count goes from 9 to 10 (another multiple of 5), darkness_level increments again from 2 to 3. The formula is: `if turn_count % 5 == 0 AND turn_count > 0 → darkness_level += 1`.

---

## TEST-017: Crypt materializes when darkness_level reaches 5

**Given:**
- `STATE.json:world.darkness_level = 4`
- `STATE.json:world.flags.crypt_materialized = false`
- `STATE.json:world.turn_count = 19` (about to hit 20 — next multiple of 5)
- `WORLD/crypt.md` does NOT exist

**When:** Player takes a valid action — turn_count becomes 20 → darkness check fires → darkness_level becomes 5.

**Then:**
- `STATE.json:world.darkness_level = 5` ✓
- `STATE.json:world.flags.crypt_materialized = true` ✓
- `WORLD/crypt.md` now EXISTS with full room description, Bone Amulet item, west exit ✓
- `WORLD/REGISTRY.json` great_hall exits now include `"east": { "to": "crypt", "requires_darkness_min": 5 }` ✓
- `WORLD/great_hall.md` Exits section now includes "**East** → The Crypt" ✓
- `LOGS/session.md` has entry describing the castle shifting and new passage appearing ✓

**Expected narration:** "A grinding sound reverberates through the Great Hall. Where solid stone stood, a dark passage breathes open to the east."

**Important:** The crypt materialization happens as part of the darkness check, which runs AFTER turn_count is incremented. The player's move still completes normally first.

---

## TEST-018: Game over when darkness_level reaches 10

**Given:**
- `STATE.json:world.darkness_level = 9`
- `STATE.json:world.turn_count = 44`
- `STATE.json:player.status = "alive"`

**When:** Player takes any valid action — turn_count becomes 45 → darkness check fires → darkness_level becomes 10.

**Then:**
- `STATE.json:world.darkness_level = 10` ✓
- `STATE.json:world.turn_count = 45` ✓
- `STATE.json:player.status = "consumed"` ✓
- `PLAYER.md` Status field = "consumed" ✓
- `LOGS/session.md` final entry reads: "THE CASTLE HAS CONSUMED YOU." ✓

**Expected narration:** "The darkness is absolute. The castle has taken you. You can no longer move, speak, or think. GAME OVER. Type **restart** to begin again."

**Post-game state:** No further state changes are possible. All subsequent commands are rejected with the consumed message until the player says "restart".

**Note on sequence:** The player's valid action executes first (e.g., they moved to a new room), THEN turn_count increments, THEN darkness check runs and triggers game over. The move itself succeeded — the darkness claimed them after they arrived.
