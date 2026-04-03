# Command: TAKE

**Triggers:** take, grab, pick up, get, collect, retrieve

---

## Resolution Algorithm

### Step 1 — Parse item
Extract item reference from player input.
Resolve to canonical item ID using CLAUDE.md §Item Name Resolution.

### Step 2 — Check item presence
For the resolved item ID, check `STATE.json:items.<id>`:
- `location == STATE.json:player.current_room` AND `taken == false`

**If item not found in current room:**
- Respond: "There is no [item name] here."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] TAKE {item} → not present in {room}`
- Stop.

**If item found:**
Continue to Step 3.

### Step 3 — Execute pickup

1. `STATE.json:items.<id>.location = "inventory"`
2. `STATE.json:items.<id>.taken = true`
3. Add `<id>` to `STATE.json:player.inventory`
4. `STATE.json:world.turn_count += 1`
5. Write `STATE.json`
6. Write `PLAYER.md` (update inventory)
7. Write `INVENTORY/items.md` (add item)
8. Write `WORLD/<current_room>.md` — remove item from "Items Here" section
   - Find the bullet line starting with `- **[Item Name]**` and delete it
   - If the "Items Here" section is now empty, replace its contents with `*(none)*`
9. Append to LOGS: `[Turn {N}] TAKE {item_id} → picked up, now in inventory`
10. Run darkness check
11. Narrate pickup using item-specific flavor text (see below)

---

## Pickup Flavor Text

| Item ID | Narration |
|---------|-----------|
| `torch` | "You take the torch from its sconce. Its warmth is faint comfort in the castle's chill." |
| `iron_key` | "The key is cold and heavy. Something about its weight feels like obligation." |
| `rusty_sword` | "You take the sword. Rust flakes off onto your palm like dry blood. Still, it has an edge." |
| `ancient_scroll` | "The moment your fingers close around it, the hum intensifies. Whatever is sealed inside stirs." |
| `bone_amulet` | "You lift the amulet. It's unpleasant to touch — like holding something that knows it should have stayed buried. But immediately, the shadows feel slightly less aggressive." |

---

## WORLD File Update — Detail

When removing an item from a room file, find the "Items Here" section and delete the item's bullet point:

**Before:**
```
## Items Here

- **Torch** — A guttering torch hangs in a rusted sconce near the gate. Its flame is weak but persistent.
```

**After:**
```
## Items Here

*(none)*
```

If there were multiple items and only one is taken, remove only that item's bullet.
