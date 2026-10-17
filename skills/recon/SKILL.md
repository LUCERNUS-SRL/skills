---
name: recon
description: Recon how large codebases on your stack solve a question, and get up to 3 approaches, each with the repos that use it.
disable-model-invocation: true
---

Answer "how do large codebases on our stack do this?" from their code. A run surveys 10 to 15 real codebases that face the same problem and reports up to 3 **approaches**, with the repos behind each.

Every claim is one of three kinds: **observed** in a repo's code (cited by permalink), **stated** in the repo's own docs, such as its README, ADRs, CONTRIBUTING or design notes (cited by link), or **inferred** by you, and labelled so.

1. **Sharpen** the question until a repo can answer it with code: name the thing to look for (a folder layout, a module boundary, a call path, a config). Take the stack from the user's argument, else infer it from this repo's manifests: the language plus the framework that shapes the question, major version included. A library is part of the stack only when the question is about it. Read how this repo handles the question today, and name its **forces**: what makes the question hard here, such as many features sharing one model, several teams, multi-tenancy, offline sync or third-party plugins. Done when the question names something findable in a repo, and you know the stack, the forces and this repo's current approach.
2. **Shortlist** 10 to 15 repos plus up to 5 reserves, from `gh search repos`, the flagship projects you know on that stack, and any repo the user names. A repo qualifies when it:
   - faces the same forces, judged from its README and layout (being large or busy is no evidence of that);
   - is a production application or a major product, never a starter, template, tutorial, demo or example;
   - was pushed to in the last 12 months (`gh repo view {repo} --json pushedAt,isTemplate,isArchived`).

   When `gh` is missing or logged out (`gh auth status`), or searches hit a rate limit, shortlist from what you have and the projects you know, and leave the activity check to Group. Without `git`, stop and tell the user. Done when every repo on the list and in the reserves meets all three, with the forces it shares noted beside it. When fewer than 10 qualify, the list is what qualified.
3. **Read** the repos. Make one temp directory (`mktemp -d`), then spawn parallel sub-agents in one message, at most 5, about 3 repos each. Give each the sharpened question, the stack, the forces, its repos, the temp directory, the three claim kinds and this brief:

   > For each repo, clone it into the temp directory with `git clone --depth 1 --filter=blob:none --sparse`. `git ls-tree -r --name-only HEAD` shows the whole layout without downloading files; `git sparse-checkout add` the paths you need. A clone that fails or runs past 2 minutes is retried once, then reported as unreadable with its error. Read at most 20 files per repo. Find how the repo handles the question, and return per repo:
   > - the date of its last commit (`git log -1 --format=%cI`);
   > - observed: its approach in one line, 1 to 3 permalinks at the cloned SHA (`https://github.com/{owner}/{repo}/blob/{sha}/{path}#L{a}-L{b}`), and one excerpt, a folder tree or at most 15 lines of code;
   > - stated: why it chose this, with a link, only when its own docs say. A reason the docs leave out stays out.
   >
   > Under 300 words.

   Replace each unreadable repo with a reserve, in one more round at most. When the harness has no sub-agents, follow the brief yourself, one repo at a time. Delete the temp directory before leaving this step, whatever happened in it. Done when every repo has a note or is unreadable with its reason, and the temp directory is gone.
4. **Group** the notes into approaches, after dropping each repo whose last commit is over 12 months old. Two repos share an approach when they make the same structural choice, whatever they call it. Order approaches by how many repos use them, then by how closely those repos share this one's forces, and keep the first 3. There is a **consensus** only when one approach covers more than half of the repos read. Done when every repo sits under a kept approach or is listed as not shown, with its reason.
5. **Report** in chat in this shape, or to a file when the user asks:

   ````
   **Recon:** {sharpened question} · {stack} · {N} codebases read
   {Consensus: {approach} ({k} of {N}). | No consensus.}

   **1. {Approach name}**
   {2-3 sentences, observed: how it works}
   ```{lang}
   {excerpt from one of its repos}
   ```
   - [{owner/repo}]({permalink}): {one observed sentence on how this repo does it}
   - ...

   Stated: {a reason one of its repos' docs give} ([source]({link}))
   Inferred: fits when {condition, in terms of this codebase}; costs {cost, in terms of this codebase}.

   **2. ...**

   **You today** (inferred): {the approach this codebase is closest to, in one line}
   **Not shown:** {owner/repo}: {fits no kept approach | unreadable: reason | inactive}, ...
   ````

   Leave out a `Stated` line no doc supports. With fewer than 5 repos read, add `thin survey` to the header line. Done when every approach has its excerpt, its repos and its inferred fits and costs, every claim carries its kind, and the approaches stand side by side: recommend one only when the user asks.
