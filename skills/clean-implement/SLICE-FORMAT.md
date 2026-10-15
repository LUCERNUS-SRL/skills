# Slice Format

A slice file holds one slice, from its cut to its commit.

## Template

```md
# {NN} {Title}

Kind: {feature | refactor | bugfix}
Status: {open | in-progress | done | cancelled}
Blocked by: {NN, NN | none}

## What to build

{The behaviour or structure this slice delivers, seen from outside the code.}

## Acceptance criteria

- [ ] {criterion}

## Plan

Seams:
- [ ] {seam}: {unit | integration | end-to-end}

Blast radius: {who observes the change, and how you searched}
Baseline: {checks already failing | none}

## Result

Commit: {short sha and subject | cancelled: why}
Follow-ups:
- {follow-up}: {slice NN | out of scope | cancelled}
```

## Example

```md
# 02 Add CSV export

Kind: feature
Status: done
Blocked by: 01

## What to build

An "Export CSV" action on the order list downloads the visible orders, one row
each, with the screen's columns.

## Acceptance criteria

- [x] `exportOrdersCsv(orders)` returns a header row and one row per order.
- [x] An empty list exports the header row only.
- [x] The file starts with a UTF-8 BOM.

## Plan

Seams:
- [x] `exportOrdersCsv()`: unit
- [x] `GET /orders/export.csv`: integration

Blast radius: only the new route calls `exportOrdersCsv`; no overrides or
serialized shapes (searched callers and string references, scripts included).
Baseline: none

## Result

Commit: 4f2a91c ENG-142 - 02 Add CSV export of orders
Follow-ups:
- Order date timezone: slice 03
```

## Rules

- **The file name** is `NN-slug.md`: `NN` is the run order from `01`, the slug short and lowercase. A slice added later takes the next free number.
- **What to build** says, by kind: the behaviour a feature makes work end to end, the structure a refactor leaves behind, or the wrong and the right behaviour of a bugfix. It names types and functions, not file paths, which go stale before the slice is built.
- **Plan and Result** stay bare headings until Build fills them.
