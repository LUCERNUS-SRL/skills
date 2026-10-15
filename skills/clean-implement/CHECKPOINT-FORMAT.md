# Checkpoint Format

The checkpoint goes to the user as one message once Refine is done.

## Template

```md
## Checkpoint: {NN} {Title}

### Delivered
- [x] {acceptance criterion}: {the test or command that proves it}

### Reading order
1. `{path}`: {what to check here}

### Decisions
{each direction decision waiting on the user, with its options}
- **{Decision name}** (default): {the detail decision taken while building}. Why: {the reason}.

### Pass log
Pass {n}: {n} fixed, {n} rejected
{the pass's findings, each ending in **fixed**, or in **rejected** and the reason}

### Follow-ups
- **{Follow-up}**: {what it is and where}. → {your pick: new slice | edit slice NN | out of scope | cancelled}
```

## Example

```md
## Checkpoint: 02 Add CSV export

### Delivered
- [x] Header row and one row per order: `exportOrdersCsv › one row per order`
- [x] An empty list exports the header row only: `exportOrdersCsv › empty list`
- [x] The file starts with a UTF-8 BOM: `GET /orders/export.csv › sends BOM`

### Reading order
1. `src/export/csv.ts`: the formatter; start here.
2. `src/export/csv.test.ts`: one test per criterion above.
3. `src/routes/orders.ts`: the new route only wires the formatter in.

### Decisions
- **Column order** (default): the screen's order. Why: support reads both side by side.

### Pass log
Pass 1: 2 fixed, 1 rejected
**src/export/csv.ts**
- **possible Long Function** in `export()`: header and rows in one body. → Split them. **fixed**
- **possible Flag Argument** in `toCsv()`: `true` adds the header. → `toCsvWithHeader`. **fixed**
**src/dates.ts**
- **possible Duplicated Code** in `formatDate()`: looks like `formatDay`. **rejected**: differs by locale.
Pass 2: 1 fixed, 0 rejected
**src/export/csv.ts**
- **possible Mysterious Name** in `writeRows()`: exposed by the pass 1 split. → `writeOrderRows`. **fixed**
Pass 3: clean

### Follow-ups
- **Order date timezone**: `orderedAt` exports in server time; support works in local time. → new slice
```

## Rules

- **Delivered** covers every acceptance criterion.
- **Reading order** covers every file in `git diff HEAD`, in the order a reviewer should read them: the entry point first, tests beside the code they prove.
- **Decisions** opens with the direction decisions waiting on the user, then the detail decisions taken as default.
- **Pass log** lists every pass, its findings exactly as the clean-code pass returned them.
- **Empty sections** stay, with `none`, so the user sees each was considered.
