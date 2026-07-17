# Agent Skills

Reusable skills for AI coding agents, distributed through the [`skills`](https://github.com/vercel-labs/skills) CLI.

## Available skills

### `write-pr-description`

Draft clear, evidence-based pull request comments for:

- Review summaries and change requests
- Implementation updates
- Responses to reviewer feedback
- Inline review comments

The skill returns a ready-to-paste draft by default. It posts a comment only when explicitly requested and when the agent has access to the pull request.

## Install

Install globally for Codex:

```bash
npx skills add Soare-Robert-Daniel/skills \
  --skill write-pr-description \
  --agent codex \
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
npx skills add . --skill write-pr-description --agent codex --global --yes
```

Confirm the installation:

```bash
npx skills list --global --agent codex
```

## Use

Invoke the skill explicitly in Codex:

```text
Use $write-pr-description to draft a review summary for this pull request.
```

You can also provide the relevant context directly:

```text
Use $write-pr-description to turn these notes into a concise PR comment:
- The null check is missing in src/user.ts:42.
- Unit tests pass.
- Request changes because this can crash for logged-out users.
```

More examples:

```text
Use $write-pr-description to write an implementation update from the current diff and test results.
```

```text
Use $write-pr-description to rewrite my response to this reviewer so it is direct and actionable.
```

```text
Use $write-pr-description to draft an inline comment for src/api.ts:88. Mark it as blocking and explain the failure mode.
```

The skill will use the pull request, diff, checks, and discussion when those are available to the agent. Otherwise, include that information in the prompt.

## Update or remove

```bash
npx skills update write-pr-description --global --yes
```

```bash
npx skills remove write-pr-description --global --agent codex --yes
```
