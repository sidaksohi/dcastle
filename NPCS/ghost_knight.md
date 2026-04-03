# Sir Aldric the Bound

**NPC ID:** `ghost_knight`
**Location:** `great_hall` *(permanent — never moves)*
**Current State:** `freed`
**Spoken To:** `true`
**Quest Complete:** `true`

---

## Description

Sir Aldric was the last knight of the castle before the curse fell. He died at the gate — or believed he did. In truth, the darkness pulled him back before he could fully leave, shackling his soul to the Great Hall with chains of cold light.

He appears as a figure in dissolving plate armor, translucent at the edges, the chains of pale light wound through his chest and wrists and ankles, pulsing in rhythm with the castle's hunger. His eyes are hollow lanterns. His voice sounds like wind through ruined stone.

He has been waiting here for a very long time. He knows what you are. He knows what you need to do.

---

## State Machine

```
[bound / spoken_to: false]
    ──TALK──► [bound / spoken_to: true]    (First Encounter)

[bound / spoken_to: true]
    ──TALK──► [bound / spoken_to: true]    (Repeat Plea)
    ──USE ancient_scroll──► [freed / quest_complete: true]

[freed / quest_complete: true]
    ──TALK──► [freed]                      (Gratitude — no state change)
```

---

## Dialogue

### First Encounter
*(spoken_to: false → true)*

> *The figure turns. Two pale lights where eyes should be regard you without warmth or welcome.*
>
> "Another one comes to the castle. And like all the others, you will linger too long."
>
> *The chains pulse.*
>
> "I am Aldric. I was a knight. Now I am... this. The curse does not kill outright — that would be mercy. It *collects*. It is patient. It waits for you to stop moving."
>
> *He looks at his own bound hands.*
>
> "There is a way to break what was done to me. Somewhere in this castle, sealed away by the one who cursed it — a scroll marked with black wax and a broken sigil. I have felt it for decades. It is above, I think. In the tower."
>
> *His lantern-eyes find yours.*
>
> "Bring it to me. Before the darkness reaches ten. Because when it does — there will be two of us wearing these chains."

---

### Repeat Plea
*(spoken_to: true, state: bound)*

> "Still here. Still searching. That is good."
>
> *The chains tighten slightly, then ease.*
>
> "The scroll. Black wax, broken sigil. The tower. Hurry."

---

### Gratitude (After Being Freed)
*(state: freed — triggered after quest complete)*

> *The scroll touches his hands. For a moment, nothing happens.*
>
> *Then the chains shatter — not with a sound but with a silence, an absence of the pale light that has defined this hall for centuries. Sir Aldric stands straight for the first time. He is still a ghost. He will always be a ghost. But his eyes are bright now.*
>
> "It is done. You have broken what I could not."
>
> *He looks at his hands — clean of chains, clean of light, just the ghost of a knight's hands.*
>
> "The gate is open. The darkness retreats. Go."
>
> *He gestures toward the south.*
>
> "Remember this, Wanderer: the castle is patient. But so can you be."
>
> *He begins to fade. Slowly. Peacefully. The way things should end.*

---

## Additional Dialogue Notes

### If player has rusty_sword during first encounter
After the main first-encounter speech, add:
> *His gaze drops to the blade at your side.* "That sword... I remember it. From the armory, from when men still kept it. Carry it well, Wanderer — even rust has its purpose."

### If player tries to attack Sir Aldric (use sword on ghost)
> "The blade passes through him like smoke through a keyhole. Sir Aldric regards you with an expression that is not quite disappointment and not quite patience — something between the two."
> "You cannot harm what the castle has already claimed."

### If player tries to take Sir Aldric's chains
> "The chains are not yours to take. They are not his, either. They belong to the castle."
> No state change.
