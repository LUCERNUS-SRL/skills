# Ticket Format

`ticket.md` carries a ticket between sessions. It sits beside the slices in a folder git ignores:

```
.scratch/{ticket}/
├── .gitignore    contains *, so git ignores the folder, itself included
├── ticket.md
└── slices/       one file per slice, in the shape of SLICE-FORMAT.md
```

## Template

```md
# {Ticket key}: {Title}

Source: {tracker link | file path | conversation}
Branch: {the branch every slice commits to}
Checks: {the commands that build, type check, lint and test}

## Goal

{What is true once the whole ticket is done, and who it is for, in one or two lines.}

## Brief

{The brief the user confirmed, word for word.}

## Skills

Questions: {skill | none}
Domain docs: {skill | none}
Test-first: {skill | none}

## Decisions

- **{Decision name}** ({you | default}): {the decision}. Why: {the reason}.

## Not yet sliced

- {the work}: {what it waits on}

## Out of scope

- {the work}: {why}
```

## Example

```md
# ENG-142: Export orders as CSV

Source: https://linear.app/acme/issue/ENG-142
Branch: eng-142-csv-export
Checks: npm run build && npm run lint && npm test

## Goal

Support staff can download any order list as a file that opens cleanly in Excel.

## Brief

Support answers refund requests by copying order lists out of the admin screen
by hand. Today `OrderList` renders orders only as HTML, and the one export,
`exportOrdersJson`, returns JSON nobody in support can open. The ticket asks for
CSV with the screen's columns; it leaves the encoding and the empty case open.

## Skills

Questions: grilling
Domain docs: none
Test-first: tdd

## Decisions

- **CSV encoding** (you): UTF-8 with BOM. Why: support opens every file in Excel.
- **Empty export** (default): header row only. Why: matches the JSON export.

## Not yet sliced

- Scheduled exports: waits on whether support wants them by email.

## Out of scope

- PDF export: nobody asked for it.
```

## Rules

- **The folder** is named for the ticket key, else a short lowercase slug of the title.
- **Branch** is the current branch, or a new one you create from it when the current branch is the default branch.
- **Checks** are the commands the project already runs. When you find none, ask the user.
- **Skills** names one skill per role, or `none`: Questions grills the user, Domain docs keeps the glossary and ADRs, Test-first drives the draft.
- **Decisions** are named for what they decide ("CSV encoding", not "Which encoding?"), and marked `you` when the user chose or answered, `default` when they took your recommendation. A decision the Domain docs skill recorded in an ADR or the glossary links there instead of restating it.
- **Empty sections** stay, with `none`.
