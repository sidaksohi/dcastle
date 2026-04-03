# Command: TALK

**Triggers:** talk, speak, ask, greet, address, call out to, say to, say hello, converse

---

## Resolution Algorithm

### Step 1 — Check for NPC in room
Check `STATE.json:npcs` for any NPC where `location == STATE.json:player.current_room`.

**If no NPC in current room:**
- Respond: "There is no one here to speak with. The castle offers only silence."
- No state change. No turn consumed.
- Append to LOGS: `[invalid] TALK → no NPC in {room}`
- Stop.

### Step 2 — Identify NPC and load dialogue
Read `NPCS/<npc_id>.md`.
Determine dialogue state based on `STATE.json:npcs.<npc_id>`:

**For ghost_knight:**

| state | spoken_to | Dialogue to deliver |
|-------|-----------|---------------------|
| bound | false | First Encounter dialogue |
| bound | true | Repeat Plea dialogue |
| freed | any | Gratitude dialogue |

### Step 3 — Execute talk

1. `STATE.json:npcs.ghost_knight.spoken_to = true` (set true regardless of current value)
2. `STATE.json:world.turn_count += 1`
3. Write `STATE.json`
4. Write `PLAYER.md`
5. Update `NPCS/ghost_knight.md`: change `**Spoken To:** false` → `**Spoken To:** true` (if it was false)
6. Append to LOGS: `[Turn {N}] TALK ghost_knight → {dialogue_type} delivered`
7. Run darkness check
8. Deliver the appropriate dialogue from NPCS/ghost_knight.md

---

## Sword Interaction

If player carries `rusty_sword` AND talks to ghost_knight for the first time:
Add this line to Sir Aldric's first-encounter dialogue after his main speech:
> *His gaze drops to the sword at your side.* "That blade... I remember it. From better days. Carry it well, Wanderer — even rust has its purpose."

---

## Give-Item Branch

If player says "give [item] to [NPC]":
- Delegate entirely to COMMANDS/use.md
- This is equivalent to "use [item]" with the NPC as context

---

## Repeated Talk

Talking to Sir Aldric multiple times (spoken_to: true, state: bound) costs a turn each time.
He repeats his plea. The player learns nothing new, but the darkness grows.
This is intentional — it rewards players who listen once and act, not those who stall.
