# dcastle Test Suite

Tests are structured Markdown scenarios using **Given / When / Then** format.

The AI engine is the test runner. No code executes. The AI simulates each scenario and verifies file state.

---

## How to Run Tests

Say any of these to the AI:

| Command | What runs |
|---|---|
| `run tests` | All 18 scenarios across all 5 files |
| `run tests/test_movement.md` | Movement scenarios only (TEST-001 to TEST-005) |
| `run tests/test_inventory.md` | Inventory scenarios only (TEST-006 to TEST-009) |
| `run tests/test_npc.md` | NPC scenarios only (TEST-010 to TEST-012) |
| `run tests/test_state_consistency.md` | Consistency scenarios only (TEST-013 to TEST-015) |
| `run tests/test_darkness.md` | Darkness/evolution scenarios only (TEST-016 to TEST-018) |

---

## What the AI Does When Running Tests

1. Reads the test file
2. For each scenario:
   - Loads the **Given** state into a simulation (does NOT modify STATE.json)
   - Applies the **When** command following SYSTEM.md rules exactly
   - Checks every **Then** assertion against the simulated file state
3. Reports **PASS** or **FAIL** per test with evidence
4. After all tests: reports summary (e.g., `18/18 PASSING`)

**Important:** Test simulation does not modify any game files. STATE.json is unchanged after a test run.

---

## Test Files

| File | Tests | What it covers |
|---|---|---|
| `test_movement.md` | TEST-001 to TEST-005 | Valid movement, blocked exits, locked doors, darkness gates |
| `test_inventory.md` | TEST-006 to TEST-009 | Item pickup, missing items, use without ownership, key use |
| `test_npc.md` | TEST-010 to TEST-012 | NPC dialogue, remote talk failure, quest completion |
| `test_state_consistency.md` | TEST-013 to TEST-015 | STATE↔PLAYER sync, inventory mirror, append-only log |
| `test_darkness.md` | TEST-016 to TEST-018 | Darkness tick, crypt materialization, game over condition |

---

## Pass Criteria

All 18 tests must PASS before the game is considered complete and playable.

A test FAILS if:
- Any **Then** assertion does not match the simulated state
- A file that should be updated was not updated
- A file that should be unchanged was changed
- STATE.json and its mirror files disagree

---

## Notation

- `STATE.json:path.to.field` refers to a specific JSON path in STATE.json
- `WORLD/<room>.md` refers to the markdown file for a specific room
- "turn consumed" = `world.turn_count` incremented by 1
- "no turn consumed" = `world.turn_count` unchanged
