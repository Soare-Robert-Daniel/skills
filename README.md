# Agent Skills

Reusable skills for AI coding agents, distributed through the [`skills`](https://github.com/vercel-labs/skills) CLI.

## Available skills

### `write-pr-description`

Draft clear, evidence-based pull request descriptions with:

- A concise summary of what changed
- Optional architecture or flow diagrams
- Data transformation notes (before/after when relevant)
- QA and verification steps
- Linked issue and PR metadata

The skill returns a ready-to-paste PR description.

## Install

Install globally for Codex:

```bash
npx skills add Soare-Robert-Daniel/skills \
  --skill write-pr-description \
  --global \
  --yes
```

Install in the current project instead:

```bash
npx skills add Soare-Robert-Daniel/skills \
  --skill write-pr-description \
  --agent codex \
  --yes
```

Install from a local checkout while developing the skill:

```bash
git clone https://github.com/Soare-Robert-Daniel/skills.git
cd skills
npx skills add . --skill write-pr-description --global --yes
```

Confirm the installation:

```bash
npx skills list --global
```

## Use

Invoke the skill explicitly in Codex:

```text
Use $write-pr-description to draft a complete pull request description for this diff.
```

You can also provide the relevant context directly:

```text
Use $write-pr-description to turn these notes into a concise PR description:
- The null check is missing in src/user.ts:42.
- Unit tests pass.
- Add a short QA section with exact reproduction and verification steps.
```

More examples:

```text
Use $write-pr-description to write a PR description from the current diff and test results.
```

```text
Use $write-pr-description to include a Mermaid diagram only if it clarifies the architecture changes.
```

```text
Use $write-pr-description to include data transformation before/after examples and link related issues.
```

The skill will use the pull request, diff, and checks when those are available to the agent. Otherwise, include that information in the prompt.

## Update or remove

```bash
npx skills update write-pr-description --global --yes
```

```bash
npx skills remove write-pr-description --global --yes
```
