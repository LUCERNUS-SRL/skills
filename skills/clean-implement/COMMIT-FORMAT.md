# Commit Format

The message of the commit Finish makes.

## Template

```
{Ticket key} - {NN} {what the slice does, in the imperative}

{Why: one to three lines on the behaviour and the reason for it.}

Decisions: {decision names | none}
Refined: {n} passes, {n} fixes
Follow-ups: {follow-up names | none}
```

## Example

```
ENG-142 - 02 Add CSV export of orders

Support needs order lists they can open in Excel; the JSON export
forces them through a converter.

Decisions: CSV encoding, Empty export, Column order
Refined: 3 passes, 3 fixes
Follow-ups: Order date timezone
```

## Rules

- **The subject** stays under 72 characters.
- **The repo's own commit convention**, when it has one, replaces the subject's shape, keeping the ticket key and slice number.
- **Decisions and Follow-ups** use the names from `ticket.md` and the checkpoint, so a reader can look each one up.
