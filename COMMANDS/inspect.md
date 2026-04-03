# Command: INSPECT

**Triggers:** look, examine, inspect, describe, where am i, what's here, survey, scan, l

**Turn cost: 0 (free action)**

---

## Resolution Algorithm

### Step 1 — Identify current room
Read `STATE.json:player.current_room` → room ID.

### Step 2 — Build room report

**Base description:** Read `WORLD/<current_room>.md` §Description. Use verbatim.

**Items present (dynamic):**
- For each item in `STATE.json:items`:
  - If `items.<id>.location == current_room` AND `items.<id>.taken == false`
  - Include it in the items list with its name and description
- If no items: "Nothing of note lies here."
- If player carries `torch` (torch in inventory): add one extra atmospheric sensory detail
  - Example extra lines by room:
    - entrance: "By torchlight, you notice deep scratches in the gate — someone tried to claw their way out."
    - great_hall: "The torchlight catches Sir Aldric's chains. They are not iron — they are light, somehow solid."
    - armory: "By torchlight, you notice bloodstains on the floor behind the weapon rack."
    - dungeon_cell: "The torch illuminates crude tally marks scratched into the wall. You count them. You stop counting."
    - tower: "By torchlight, you notice the pedestal has an inscription: *'For the one who returns.'"*
    - crypt: "The torchlight makes the carved faces on the sarcophagi seem to shift."

**NPCs present:**
- For each NPC in `STATE.json:npcs`:
  - If `npcs.<id>.location == current_room`
  - Describe NPC based on their current state:
    - ghost_knight, state=bound: "Sir Aldric the Bound drifts here, chained by pale light."
    - ghost_knight, state=freed: "Sir Aldric stands free, his light no longer the light of chains."

**Exits available:**
- For each exit in `WORLD/REGISTRY.json:rooms.<current_room>.exits`:
  - Evaluate conditions against current STATE.json
  - If all conditions met: list the direction
  - If conditions NOT met: do NOT list or hint at the exit
  - Exception: if `requires_item` blocks an exit, you MAY describe the obstacle thematically without naming the item needed
    - "To the north, a rusted iron door stands sealed."

### Step 3 — Format response

```
[Room Name]

[Base description — 2-4 sentences]

[Torch detail if applicable — 1 sentence]

Items: [bulleted list or "Nothing of note."]
NPCs: [bulleted list or none]
Exits: [bulleted list of available directions with destination names]
```

### Step 4 — Log and narrate

- Append to LOGS: `[free] INSPECT {room_id} → described`
- Do NOT increment `turn_count`
- Narrate the formatted report

---

## Note

INSPECT is the safest command. It never changes state, never costs a turn, and cannot fail. Encourage players to use it freely.
