# Session Log

> **Append-only.** The engine writes one entry per command. Existing entries are never modified or deleted.
> This log is the permanent record of everything that has happened in dcastle.

---

## Entry Format

```
[Turn N] COMMAND → RESULT
Narrative note (optional, 1 sentence)
```

Free actions (INSPECT) use `[free]` instead of a turn number.
Invalid/failed actions use `[invalid]` and note the reason.

---

## Log

```
[Turn 0] GAME START
The Wanderer enters the castle. The great gate seals behind them with the finality of a stone lid.
Darkness Level: 1. The curse is patient. The clock has begun.

[free] INSPECT entrance → described

[Turn 1] MOVE east → arrived in Dungeon Cell

[invalid] MOVE north → blocked: iron_key required, not in inventory

[invalid] USE iron_key → not in inventory

[Turn 2] TAKE iron_key → picked up from dungeon_cell, now in inventory
[Turn 3] MOVE north (dungeon_cell) → iron_key consumed, door unlocked, arrived in Great Hall

[Turn 4] TALK ghost_knight → first encounter delivered. Attempted to free him — no scroll yet. Sir Aldric directs player to armory (west) and tower (up).
The darkness deepens slightly. Sir Aldric's chains pulse.

[Turn 5] MOVE west → arrived in Armory. AUTO-PICKUP: rusty_sword.
⚠ DARKNESS TICK: turn 5 → darkness_level 1→2. The shadows lengthen.

[Turn 6] MOVE east → returned to Great Hall. No items.

[Turn 7] MOVE up → arrived in Watchtower. AUTO-PICKUP: ancient_scroll.
The scroll hums the moment it is touched.

[invalid] USE ancient_scroll → ghost_knight not in current room (tower). Player must return to great_hall.

[Turn 8] MOVE down (tower) → arrived in Great Hall. No items.

[Turn 9] QUEST COMPLETE — ancient_scroll given to Sir Aldric.
  ghost_knight.state: bound → freed | quest_complete: true
  darkness_level: 2 → 0 (curse shattered)
  flags.gate_unlocked: true
  ancient_scroll: inventory → used
THE CASTLE HAS BEEN BROKEN. THE GATE IS OPEN.

[Turn 10] MOVE south → returned to Castle Entrance.
⚠ DARKNESS TICK: turn 10 → darkness_level 0→1. The castle stirs again, already hungry.

[Turn 11] MOVE south (entrance) → flags.gate_unlocked=true → ESCAPE
═══════════════════════════════════════════════
THE WANDERER HAS ESCAPED THE CASTLE.
═══════════════════════════════════════════════
player.status: alive → escaped
Turns taken: 11 | Final darkness: 1
```
