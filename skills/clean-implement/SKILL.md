---
name: clean-implement
description: Implement one slice of a sliced unit, clean and approved step by step.
disable-model-invocation: true
---

A **slice** is one increment of a unit, written by `clean-slice` into `.scratch/<unit>/` (`UNIT.md` plus `NN-title.md` files). One slice runs per session. Fill the headings already in the slice file; keep its shape.

A **seam** is a place where behaviour can be tested, such as a public function. A **suggestion** is a note for the user, never an action.

The **checks** are whatever the project already runs: build, type check, linter, tests. When none can be found, ask the user to name the command; without one, stop. A **move** is one baby step followed by the checks. **Red** is a check that passed at the baseline and now fails. On red, revert the move and retake it smaller; after 2 reverts of the same move, stop and show the user the failing output and what you tried.

Ask open questions in rounds: number each question and give your recommended answer. A question without an explicit answer stays open; wait for the round before moving on.

Take the unit from the user's argument, else the only folder in `.scratch/`, else ask. With no state, a change to one function that fits one commit runs without it: Plan in chat, then Build, Clean and Checkpoint. Anything larger needs `clean-slice` first.

## Steps

1. **Verify** before touching anything: the branch matches `Branch`, every `done` slice's commits sit in `git log <Base>..`, `git status` is clean or the user names the dirty files, and the source has not changed. On any mismatch show it and stop, changing nothing.
2. **Plan** with the user, in one stop. A `doing` slice resumes with its `Plan` and ticked seams; otherwise propose the lowest-numbered `todo` slice whose blockers are done or dropped. Settle:
   - the seams, the cycle order, and any change to a public name or signature or move between modules;
   - the blast radius of every public or protected member the slice changes or removes, and of every behaviour change under an unchanged signature: callers, overrides and implementers, serialized shapes, config and string references, other repos (grep, type hierarchy, non-code files);
   - the behaviour for each edge case the slice decides;
   - the checks, with exact commands and scope including dependents, and the baseline as failing test ids (rerun a failure once; a flip is flaky).

   Write it under `Plan` and `Blast radius`, and set the slice `doing` on the user's go. Done when every `Done when` item maps to a seam, every consumer has a covering test or an [Ask first](#ask-first) line, and the user says go.
3. **Build** the draft by kind, with `clean-code` loaded so the draft is written to its rules. If a test-driven development skill is available, use it for the loops. Commit on the unit branch, never on the default branch; push nothing.
   - Feature: a red-green cycle per seam.
   - Refactor: characterization tests first for any behaviour moved without them, then structure only, in moves.
   - Bugfix: a failing test first, then the smallest fix, covering the base and each override.

   Commit each green cycle and tick its seam; red is never committed. A `Done when` that proves wrong stops the slice: propose the corrected text, change it on approval, and record it under `Decisions`. Work found beyond the slice is an Ask first item. A refactor found necessary mid-slice is the exception: commit the green work, commit the rest as a WIP commit named under `WIP`, set this slice `todo`, add the refactor as a slice that blocks this one, and stop for the user to approve it. Done when every seam is ticked, green and committed.
4. **Clean** the draft against `clean-code`.
   - Scope: every function the slice touches, in full (Boy Scout Rule: leave touched code cleaner), plus the class-level smells of any class that gained a member. A refactor slice's scope is the code it names; a bugfix slice's, the fixed function. Scope is hand-written code that survives the unit. A function in a later `todo` slice's `Touches` is cleaned only for what this slice changed. Cleanup larger than the slice is a suggestion.
   - Cleanup keeps what existing callers observe (returns, errors, ordering, logs). A touched function without tests gets a characterization test first, or becomes a suggestion.
   - Correctness pass: retry and partial-failure behaviour, concurrency, security, and tests on the error paths.
   - Review the scope yourself with the `clean-code` smells. When it is large (over ~100 lines, 3 or more files, or a public API) and the harness can spawn sub-agents, also give a fresh-context one the scope and the diff, without the plan, to review and report findings.
   - Accept or reject each finding, with a one-line reason per rejection; a rejected finding is not raised again. Fix the accepted ones in moves. Later passes review the lines the last fix changed and the call sites of anything whose signature or behaviour changed. At most 3 passes; leftovers become suggestions.

   Done when every function in scope was checked against every applicable smell and the last pass yielded no accepted finding, or the cap was reached.
5. **Checkpoint** in this order. Needs a decision: Ask first items, rejected findings, unpinned code, each with your recommendation. Delivered: each `Done when` item with its proof, and the cleanup. Then suggestions, baseline failures and smells outside the scope. The user approves or rejects each decision item; an approved work item becomes a new slice (next free number, `Blocked by` set) or an `Out of scope` line, as the user chooses. Apply approved cleanup and revert rejected cleanup in moves.
6. **Finish**: commit the build, then the clean-up as a separate `refactor:` commit, each subject `<unit> NN: <what>`, the body carrying decisions and what was left out. Fill `Commits` and `Result`, set the slice `done`, add its decisions to `UNIT.md` (one that outlives the unit also goes to the ticket, an ADR or `CONTEXT.md`), and tell the user to start the next slice in a fresh session. After the last slice, offer to delete the unit's folder.

Done when the user has approved the checkpoint, the checks show no failures beyond the baseline, and the slice is committed and marked.

## Ask first

Collect these as you work and put them to the user at the checkpoint, together. Only items that are new and unrequested belong here.

- A change to a public name or signature, a move between modules, or a data, migration or contract change found mid-build: stop and ask at once.
- Deleting an exported or public member found dead, after a repo-wide search that included non-code files.
- Code with no tests, where pinning its behaviour would outgrow the slice.
- A bug found: its location, what is wrong, and a suggested fix, for the user to slice.
- Work found beyond the slice.
