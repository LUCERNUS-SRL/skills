---
name: recon
description: Recon how large codebases on your stack solve a question, and get up to 3 approaches, each with the repos that use it.
disable-model-invocation: true
---

Answer "how do large codebases on our stack do this?" from their code. A run reads 10 to 15 **repos** that face the same problem as the **project** (the codebase you run in), and reports up to 3 **approaches**, each with the repos that take it. Two repos take the same approach when they make the same structural choice, whatever they call it.

Take the question from the user's argument, else from the conversation, and ask for it only when neither has one. From there you decide alone, up to the report: its header shows what you chose (the question, the stack, the constraints), so the user can correct it and run again.

1. **Sharpen** the question until a repo can answer it with code: name the thing to look for (a folder layout, a module boundary, a call path, a config). The **stack** is the one the user's argument names, else the language and the framework that shape the question, major version included, read from the manifests of the part of the project it touches. A library is part of the stack only when the question is about it. Read how the project handles the question today (`none` when it has no code for it yet), and name its **constraints**: what makes the question hard here, such as many features sharing one model, several teams, multi-tenancy, offline sync or third-party plugins. Done when the question names something findable, and you know the stack, the constraints and the project's current approach.
2. **Shortlist** 10 to 15 repos from `gh search repos` and the flagship projects you know on the stack. A repo qualifies when it:
   - shares the project's constraints, judged from its README and layout (size and stars say nothing about that);
   - is a production application or a major product, rather than a starter, template, tutorial, demo or example;
   - was pushed to in the last 12 months and is not archived (`gh repo view {repo} --json pushedAt,isTemplate,isArchived`).

   A repo the user names goes on the list as given; the project itself stays off it. This step and the next need `git` and a logged-in `gh` (`gh auth status`): when either is missing, stop and tell the user which. Done when every repo on the list qualifies, with the constraints it shares noted beside it. When fewer than 10 qualify, the list is what qualified.
3. **Read** the repos. Make one temp directory (`mktemp -d`), then spawn parallel sub-agents in one message, at most 5, about 3 repos each. Give each the question, the stack, the constraints, its repos, the temp directory, the steps below and [NOTE-FORMAT.md](NOTE-FORMAT.md). When the harness has no sub-agents, take the steps yourself, one repo at a time. For each repo:
   - Clone it into the temp directory with `git clone --depth 1 --filter=blob:none --sparse`. When a git command fails on the repo, or its clone runs past 2 minutes, the repo's note is the error.
   - List its layout with `git ls-tree -r --name-only HEAD`, and `git sparse-checkout add` only the paths you read, at most 20 files.
   - Write its note in the shape of [NOTE-FORMAT.md](NOTE-FORMAT.md).

   Done when every repo has a note and the temp directory is deleted, whatever happened in it. When any note is an error, the run fails: give the user each failed repo with its error, and stop.
4. **Group** the notes into approaches. A repo is **read** when its note names an approach. Order the approaches by how many repos take them, then by how closely those repos share the project's constraints, and keep the first 3. There is a **consensus** when one approach covers more than half of the repos read. Done when every repo read sits under one approach.
5. **Report** in chat in the shape of [REPORT-FORMAT.md](REPORT-FORMAT.md), or to a file when the user asks. Done when the report is out with the approaches side by side; recommend one only when the user asks.
