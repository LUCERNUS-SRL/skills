# State templates

Write every state file from these shapes: same headings, same order, same field names. `Blocked by` holds slice numbers (`01, 03`) or `none`. `Status` holds one token. `NN-title` is the number plus a lowercase slug of at most 4 words.

## UNIT.md

```
# ENG-142: Export orders as CSV

Source: <link | path | "pasted below">
Base: <sha the unit started from>
Branch: <unit branch>

## Goal
<2 or 3 lines: the outcome and who benefits>

## Out of scope
- <item> (<why>)

## Slices
01 pin-export-behaviour: done
02 add-csv-export: doing

## Decisions
- 01: <decision>. Why: <reason>. Outlives the unit: yes (ADR 0007) | no
```

A decision outlives the unit when a future reader of the code would otherwise ask why.

## NN-title.md

```
# 02 add-csv-export

Kind: feature | refactor | bugfix
Status: todo | doing | done | dropped
Blocked by: 01
Touches: <functions and modules this slice changes>
Commits: <short sha and subject per commit>

## Delivers
<one behaviour or one structural change>

## Done when
- [ ] <checkable item>

## Plan
Seams:
- [ ] <seam>: <test level>
Cycle order: <seam order>
Signature and module changes (approved): none | <list>
Checks: <exact commands and scope, including dependents>
Baseline: <failing test ids and reason> | none
WIP: none | <commit holding unfinished work>

## Blast radius
Callers: <found how, how many>
Overrides and implementers: <list | none>
Consumers (serialized shapes, config, strings, other repos): <list | none>
Coverage: <each consumer: test name | listed under Ask first>

## Result
Done when evidence: <item: test or command that proves it>
Tests added: <name per seam>
Left out: <item: new slice | Out of scope | rejected, with reason>
Failures beyond baseline: none
```

`Result` is written at Finish. `Why dropped` goes under `Left out` for a dropped slice.
