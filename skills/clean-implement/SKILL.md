---
name: clean-implement
description: Slice a ticket, then build one slice per session, test-first, refined in passes and approved by you, as one commit each.
disable-model-invocation: true
---

Turn a **ticket** (a tracker issue, a spec, pasted text or this conversation) into clean commits, one **slice** at a time. The build follows Robert C. Martin's rule: the **first draft** is never clean code. It makes the tests pass, then **successive refinement** makes it right, one pass at a time.

The user stays in the middle: they choose the direction, approve each plan, and review each slice at its checkpoint before it is committed.

There are two modes. When `.scratch/{ticket}/ticket.md` exists, [build a slice](#build-a-slice); otherwise [slice the ticket](#slice-a-ticket). Take the ticket from the user's argument, else the only folder in `.scratch/` holding a `ticket.md`, else ask. One session runs one mode, and builds at most one slice.

## Slice a ticket

A slice is a **tracer bullet**: a narrow path through every layer it needs, green and shippable on its own, small enough for one session and one commit. This mode ends with the state written; the code stays as it is.

1. **Read** the ticket (fetch it when it is a link or a path) and the code it touches. Done when you can say how the code handles the ticket's case today.
2. **Brief** the user in one screen, in plain words, for a reader who has seen neither the ticket nor the code:
   - the problem, and who has it;
   - how the code handles it today, through the few places that matter;
   - what the ticket decides, and what it leaves open;
   - the skill for each role in `ticket.md`'s `Skills`, taken from the conversation or `AGENTS.md` when they say, otherwise proposed from the installed skills.

   The user corrects it; revise and show it again. Done when the user confirms the brief and the skills.
3. **Grill** the user with the Questions skill (`none`: grill them yourself, one round at a time), using the Domain docs skill as decisions land. On top of the Questions skill, every decision is one of two kinds:
   - A **direction** decision shapes the solution: the approach, the data model, the scope, a public name, signature or contract, a move between modules. Lay out its options evenly, and recommend only when the user asks. A decision that could be either kind is direction.
   - A **detail** decision is anything else, such as an edge case, a format or an error. It keeps the recommendation.

   Skip what the ticket or this conversation already decided. Done when every question has an answer and nothing is left assumed.
4. **Cut** the slices. A **prefactor** (a refactor that makes a later slice easy) comes first and blocks the slices that need it. A schema or API change goes **expand-contract**: the new shape lands in one slice, the old shape goes in a later one. A slice expected to touch more than 10 files or 500 changed lines is split, or kept only with the user's approval. Work you can see coming but cannot cut yet goes under `Not yet sliced`, and work the user ruled out under `Out of scope`. Show each slice's title, kind, blockers, what it builds and its acceptance criteria; the user may merge, split or reorder, and you renumber. Done when the user approves the cut, and every acceptance criterion can be checked on its own by a test or a command.
5. **Write** the state: `ticket.md` from [TICKET-FORMAT.md](TICKET-FORMAT.md), and one file per slice from [SLICE-FORMAT.md](SLICE-FORMAT.md). Done when the files are written; tell the user to run this skill again in a fresh session to build slice 01.

## Build a slice

The **checks** are the `Checks` line of `ticket.md`; the **baseline** is what fails before the slice starts. A **move** is one small edit followed by the checks. A move that stays green is staged (`git add -A`). In Refine and in a refactor slice, a move that turns a check **red** (passing at the baseline, failing now) is undone with `git restore .` and deleting any file it created, then retaken smaller. After 2 undos of the same move, stop and show the user the failing output and what you tried.

The slice lives in the working tree until Finish makes its one commit.

1. **Pick** the `in-progress` slice when there is one (a session was interrupted: resume from its ticked seams), else the **frontier**: the lowest-numbered `open` slice whose blockers are all `done` or `cancelled`. Done when the slice is chosen, the current branch is `Branch`, and the working tree holds nothing but this slice's own work. On a mismatch, show it and stop.
2. **Plan** the slice under its `Plan`, in the shape of [SLICE-FORMAT.md](SLICE-FORMAT.md):
   - the seams, the public places where its behaviour is tested, each with its test level;
   - the **blast radius** of everything the slice changes: callers, overrides and implementers, serialized shapes, config and string references, searched across the repo including non-code files;
   - the baseline: run the checks, and rerun a failure once (one that passes on the rerun is flaky).

   Show the user one screen: a recap for a reader returning cold (what this slice changes and where, what will work afterwards and how to try it, the decisions it rests on, by name), then any direction question the plan raises, then one line each for the seams, the blast radius and the baseline. The user may cancel the slice instead: set it `cancelled` with why under `Result`, and pick again. Done when every acceptance criterion maps to a seam, everything in the blast radius has a covering test or is a follow-up, and the user says go. Set the slice `in-progress`.
3. **Draft** the first draft with the Test-first skill (`none`: the rules here), seam by seam: a test that fails for the expected reason, then the least code that turns it green with no other check red, then tick the seam. The draft only has to work; making it right is Refine's job.
   - A feature runs one red-green cycle per seam.
   - A bugfix starts with a test that fails on the bug, then the smallest fix.
   - A refactor changes structure only, under tests that already pin the behaviour.

   Before changing code that has no tests, write **characterization tests** through its seams; when that would outgrow the slice, the code stays as it is and becomes a follow-up. A direction decision that surfaces mid-draft stops the draft: put it to the user at once. A detail decision takes your recommendation. A refactor this slice needs first also stops the draft: the user folds it into this slice, or makes it a new slice that blocks this one, in which case set the draft aside with `git stash push -u` and set this slice back to `open`. Done when every seam is ticked and the checks show nothing red.
4. **Refine** in at most 3 passes, each starting over on the whole result, since every fix can expose the next smell:
   - Run a `clean-code` pass yourself, its scope the slice's diff (`git diff HEAD`).
   - Fix each finding in moves, or reject it with a one-line reason. A rejected finding raised again stays rejected. A finding whose fix is a contract change becomes a direction decision at the checkpoint.
   - Log the pass, as the pass log in [CHECKPOINT-FORMAT.md](CHECKPOINT-FORMAT.md) shows.

   A slice over 300 changed lines (`git diff HEAD --shortstat`) ends with one **fresh pass** after the last: a new sub-agent, given the `clean-code` skill and the diff, never the plan, runs the pass and returns the findings, which you fix or reject and log the same way. Done when a pass ends with nothing fixed or the last pass is done, then any fresh pass is handled, with the checks showing nothing red. Findings still open then become follow-ups.
5. **Checkpoint**: show the slice in the shape of [CHECKPOINT-FORMAT.md](CHECKPOINT-FORMAT.md). The user's manual review starts here. A **follow-up** is anything found outside the slice: a bug, more work, dead public code, untested code too costly to characterize, a `Not yet sliced` item now sharp enough to cut, an `open` slice this one changed (what it builds, its acceptance criteria, its blockers). For each one the user picks a new slice, an edit to an `open` slice, a line under `Out of scope`, or cancelled; work that corrects a `done` slice is a new slice. A change the user asks for is made in moves, followed by Refine and a new checkpoint. Done when every `open` slice has been read against this one and the user approves the checkpoint.
6. **Finish**: commit the slice on `Branch`, with a message in the shape of [COMMIT-FORMAT.md](COMMIT-FORMAT.md). Fill the slice's `Result`, set it `done`, and add the slice's decisions under `Decisions` in `ticket.md`. Write each follow-up where the user put it, a new slice in the shape of [SLICE-FORMAT.md](SLICE-FORMAT.md); a `Not yet sliced` item that became a slice leaves that section. Tell the user to build the next slice in a fresh session; once no `open` slice is left, offer to delete `.scratch/{ticket}/`. Done when one commit holds the whole slice, every follow-up is written where the user put it, and the slice is `done`.
