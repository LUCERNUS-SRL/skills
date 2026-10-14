# Lucernus Skills

Agent skills published by Lucernus for Claude Code, Codex, and other agents. People install them with `npx skills add LUCERNUS-SRL/skills`, which reads this repo straight from GitHub. There is no build or release step: whatever is on `main` is what people install.

## Skill layout

Each skill is one flat folder under `skills/`:

```
skills/<name>/
├── SKILL.md            # required
├── agents/openai.yaml  # required, Codex metadata
└── ...                 # optional disclosed reference: extra .md files, scripts/
```

`SKILL.md` frontmatter carries `name` (identical to the folder name) and `description`.

### Invocation

Every skill is either **model-invoked** or **user-invoked**, and the choice shows up in two files:

| | `SKILL.md` frontmatter | `agents/openai.yaml` |
| --- | --- | --- |
| Model-invoked | `description` written for the agent, carrying its triggers | no `policy` block |
| User-invoked | `disable-model-invocation: true`, `description` is a one-line human summary | `policy.allow_implicit_invocation: false` |

`agents/openai.yaml` shape:

```yaml
interface:
  display_name: "<Title Case Name>"
  short_description: "<one line>"
policy:                             # user-invoked skills only
  allow_implicit_invocation: false
```

### Drafts

A **draft** sets `metadata.internal: true` in its frontmatter, so the installer skips it:

```yaml
metadata:
  internal: true
```

## Publishing a skill

A skill is published once it drops `metadata.internal`. In the same change, add its line to `README.md` under `## Skills`, in the **User-invoked** or **Model-invoked** list matching its frontmatter:

```markdown
- **[<name>](./skills/<name>/SKILL.md)**: <one-line summary>
```

The README lists exactly the published skills. A list's _None published yet._ placeholder goes when its first skill lands.

## Style

Punctuate with commas, colons, parentheses, or a new sentence. Em-dashes (U+2014) appear nowhere in this repo.
