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
```
