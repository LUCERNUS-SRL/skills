---
name: clean-slice
description: Slice a unit of work into reviewable increments, ready for clean-implement.
disable-model-invocation: true
---

A **unit** is the work to build: a ticket, a spec or pasted text. A **slice** is a thin vertical increment of a unit: one behaviour or one structural change, green on its own, deployable alone (schema and contract changes go expand first, contract in a later slice). The **state** in `.scratch/<unit>/` carries the unit between sessions, and `clean-implement` builds it one slice per session.

Ask open questions in rounds: number each question and give your recommended answer. A question without an explicit answer stays open: write no state and start no next round until the round is answered, then re-ask only the open ones.

Take the unit from the user's argument, else ask. Name the folder for the ticket key, else a lowercase slug of at most 4 words, and say the name in your first message.

## State

```
.scratch/
├── .gitignore       # contains *, so git ignores the state, itself included
└── ENG-142/
    ├── UNIT.md
    ├── 01-pin-export-behaviour.md
    └── 02-add-csv-export.md
```

Write each file from [state-template.md](state-template.md). `NN` is the run order; a slice added later takes the next free number, and `Blocked by` keeps it ahead of the slices that need it.

## Steps

1. **Read** the unit, the code it touches, and the source again if it is a ticket.
2. **Ask** the behaviour questions the unit leaves open (edge cases, formats, error behaviour) as the first round.
3. **Draft** the slices. A refactor that makes the change easy comes first, listed under `Blocked by` of the slices that need it. Untested code that a slice changes gets a pinning slice before it; a bugfix starts with a slice that reproduces the bug. Fill `Touches` and check the slices for overlap.
4. **Settle** the cut with the user: each slice's `Kind`, `Delivers`, `Done when`, `Blocked by` and file count (justify any over 10), plus `Out of scope`. The user may reorder; renumber to match.
5. **Write** the state with `Base` and `Branch` filled in, leaving each slice's `Plan`, `Blast radius` and `Result` as the empty headings of the template. Then tell the user to start `clean-implement` on the first slice in a fresh session.

Done when the user has approved every slice, each with a checkable `Done when`, and the state is written.
