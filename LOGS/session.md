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
```
