# dcastle

> *The castle is patient. Are you?*

An AI-native dark fantasy dungeon game where **the repository is the world**.

You don't run any code. You don't install anything. You just talk to the AI.

---

## How to Play

**Open this repository in [Cursor](https://cursor.sh) or [Claude Code](https://claude.ai/code).**

Type your commands in the chat. The AI is your game engine.

Start here:

```
look
```

---

## The Story

You are **The Wanderer** — sealed inside a cursed castle with no memory of how you arrived.

Somewhere in the dark:
- A knight has been chained here for centuries, bound by pale light
- An ancient scroll sealed with black wax holds the key to his release
- The darkness grows with every moment you linger

**Free Sir Aldric. Escape the castle. Do it before the darkness reaches 10.**

---

## Castle Map (partial)

```
         [Tower]
            │
           (down)
[Armory] ──── [Great Hall] ──── [???]
                   │
              (locked door)
                   │
[Entrance] ──── [Dungeon Cell]
     │
  [Outside?]
```

*The castle is not static. New passages appear as the darkness grows.*

---

## Commands

| What you type | What it does |
|---|---|
| `look` | Examine your surroundings |
| `go north` / `go east` / `up` / `down` | Move between rooms |
| `take torch` | Pick up an item |
| `use iron key` | Use or give an item |
| `talk to the ghost` | Speak with an NPC |
| `give scroll to the knight` | Give an item to an NPC |
| `run tests` | Run the full test suite |
| `restart` | Reset the game to the beginning |

You can speak naturally. The AI understands context.

---

## Rules

- All game state is stored in `STATE.json` — there is no hidden state
- The AI engine reads `STATE.json`, applies rules from `SYSTEM.md`, and updates files after every command
- **Darkness rises every 5 turns.** At darkness 10, the castle consumes you.
- Carrying the **Bone Amulet** slows the darkness to every 7 turns
- `LOGS/session.md` is an append-only record of everything that has happened

---

## File Reference

| File | What it contains |
|---|---|
| `STATE.json` | The source of truth — all game state |
| `PLAYER.md` | Your current status and inventory (mirror) |
| `INVENTORY/items.md` | What you're carrying (mirror) |
| `WORLD/` | The castle rooms |
| `NPCS/ghost_knight.md` | Sir Aldric the Bound |
| `LOGS/session.md` | The permanent session log |
| `TESTS/` | 18 test scenarios for game validation |
| `CLAUDE.md` | The AI engine contract |
| `SYSTEM.md` | Complete game mechanics reference |

---

## The Winning Path

There is a way to escape in as few as 8 turns. The castle will not make it easy to find.

---

## Technical

This is an **AI-native repository game**. The repository *is* the game. Files are game state. The AI is the engine.

Built with [Claude Code](https://claude.ai/code). No runtime. No dependencies. Just files and language.

> *"What enters does not leave unchanged."*
