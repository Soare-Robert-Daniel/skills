---
name: robert-pr-review
description: Review PR against Robert's standards - correctness, input handling, tests, ASD-STE100 comments, readability. Use for any code review, PR or diff feedback, branch check before merge, or "does this look OK to ship" - even without word "review".
---

# Robert's PR review

Review catches what author cannot see, says it so it gets fixed. Twenty style nits + missed unvalidated query param = failed review. Rank findings by consequence, always.

## Scope

Review diff plus enough surrounding code to judge correctness in context. Read callers of changed function; read schema before judging migration.

Pre-existing problems out of scope **unless** change makes them dangerous or is natural moment to fix. Put them in separate "Outside this change" section — author must tell what blocks merge from what is observation.

Very large diff (~1000+ changed lines): say so up front, review highest-risk files first, name files not read. Silent skim worse than partial review that admits limits.

**Check for stack before start.** Signs: base branch not trunk, body has stack chain or checklist, title has `[2/4]`-style prefix, `gh stack view` reports stack. Changes review both directions — later layer may supply what looks missing here; split itself becomes review subject.

Stacked: read `references/stacked-prs.md` first. Never guess — flagging code as unused when next layer calls it is fastest way to lose author's trust.

## Procedure

Work in order. Order stops review from turning into style pass.

### 1. Understand intent

Read PR description + linked issue before code. No description or issue link: ask for one, note as finding. Cannot judge solution without stated problem.

State intent back in one sentence. Cannot: description is first fix.

### 2. Correctness against intent

Does change do what it claims? Look for:

- Inverted logic, off-by-one, wrong at boundaries (empty collection, single element, zero, negative, null).
- Error paths that swallow failures, log-and-continue, or return success on failure.
- Resource leaks on error path — files, connections, locks, subscriptions.
- Concurrency: shared mutable state, check-then-use race, lock held across await or I/O.
- Behaviour description does not mention. Unannounced side effects: either description incomplete or change does too much.

### 3. Risk outside diff

Highest cost when missed — check explicitly:

- **Untrusted input.** Any value from user, request, file, third-party API. Must be validated at boundary; whitelist of allowed, not blacklist of forbidden. Reaches queries as bound parameter, shells as argument list not string, HTML through framework escaping.
- **Secrets.** Keys, tokens, passwords, connection strings, private hosts — in code, tests, fixtures, config, commit history. New logging must not print credentials, tokens, personal data.
- **Breaking changes.** Changed signature, response shape, field name, error code, default. What happens to existing callers, and to clients on previous version during rolling deploy?
- **Migrations / data changes.** Reversible? Locks large table? Runs before or after dependent code?
- **New dependencies.** Needed, maintained, licensed? Dependency for one helper function: question it.

### 4. Tests

Project has tests: change carries tests. Must cover new behaviour + at least one failure case, not only happy path. Test that still passes with change reverted is not test — say so.

No test setup at all: mention once as suggestion, move on. No manufactured finding.

### 5. Readability

Standard: competent engineer who never saw this code can follow it.

- Names carry meaning; same concept keeps same name throughout.
- No dead code, commented-out blocks, stray debug output, ownerless TODOs.
- No abstraction for single caller. No indirection that only adds hop.
- Functions do one thing; nesting shallow enough to read without tracking state in head.
- Follow conventions already in file. Consistency beats preference — surrounding code does it another way: defer to it, or change deliberately and say why.

### 6. Comments

Comments explain _why_, not _what_. Comment restating code = noise; comment explaining non-obvious constraint, workaround, or rejected alternative = worth more than code around it. Flag stale comments — comment contradicting code worse than none.

**Comment serves next reader, not reviewer.** Test: still makes sense to someone opening file in a year who never saw this PR? Only makes sense as author-to-reviewer message: belongs in PR description, delete from code.

Common in machine-generated changes — model narrates own reasoning into file. Flag every instance:

- **Change narration.** `// Added a null check here to fix the crash`, `// Changed this from a list to a set`. Diff + commit history already say this and stay accurate; comment rots. Six months later it describes a change nobody remembers against code that moved on.
- **Talking to reviewer.** `// As requested`, `// Per the ticket`, `// Note: I went with a map here`. Audience gone by merge.
- **Justifying obvious choice.** `// Using a dict for O(1) lookup`. Reason visible in code: comment adds line to read, nothing to know. Comment the reason reader _cannot_ infer — constraint that forced choice, not choice.
- **Narrated obviousness.** `// Loop through the users and add up their totals` above exactly that loop. Confident tone, zero information.
- **Section headers for nothing.** `// --- Validation ---` over three lines of validation, `// Helper functions` over one function.

**Should fix** when comment will read false or confusing once change is old; nit otherwise. Suggest deletion first. Propose replacement only when real _why_ worth recording — cannot name one: deletion is whole fix.

Same rule applies to you. Never rewrite bad comment into longer bad comment.

**Placement.** Comment belongs where reader hits it at moment of need, nowhere else. Three positions earn keep:

- **File or module header** — main idea. Why module exists, what it owns, the one constraint shaping design. Highest-value comment in most files, most often missing.
- **Function or docstring** — contract caller cannot see from signature: preconditions, units, ownership of returned resources, failure behaviour, safe to call twice or not.
- **Directly above single surprising line** — attached, not floating three lines up. Separated comment gets missed, drifts as code moves.

Sparse and short: 1–3 lines. Never spread one explanation across five comments in a function — say once, at highest position covering all. Reason needs paragraph: module header or linked design note, inline comment reduced to pointer.

Sparse for signal, not size. Every redundant comment lowers odds the load-bearing one gets read; every comment is claim that can go stale and lie. Three accurate comments beat thirty — but _missing_ comment costs far more than wordy one. Agent that cannot see why check exists will remove it. Never use brevity to argue deleting a comment that records a real constraint.

Comments + commit messages use ASD-STE100 Simplified Technical English. Read `references/simplified-technical-english.md` before flagging or rewriting; quote specific rule. "Use STE" alone not actionable.

STE applies to comments, docstrings, commit messages, PR description. Not identifiers, string literals, test names.

### 7. Commits and titles

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<optional scope>): <description>

<optional body>

<optional footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. Breaking change: `!` before colon (`feat(api)!: ...`) or `BREAKING CHANGE:` footer, or both.

Type must be honest. `fix` that adds feature or `chore` that changes behaviour corrupts changelog + version bump downstream. Only commit finding beyond cosmetic: section 3 found breaking change and commit not marked `!` = **blocking** — tooling ships it as minor release.

Subject line does real work:

- Say what change does, imperative — `fix(auth): reject tokens with no expiry claim`, not `fixed some token stuff` or `updates`.
- Under ~50 chars after type+scope — survives `git log --oneline` and GitHub truncation.
- No trailing period, lowercase after colon.
- Scope names real module or area, or no scope. Invented scope worse than none.

Reject activity-not-outcome subjects: `refactor: cleanup`, `chore: address review comments`, `fix: various fixes`. Someone will bisect to this commit and need to know what it did.

Body carries _why_ — constraint, bug symptom, rejected alternative. Wrap 72 chars. Issue in footer (`Refs: #431`, `Closes: #431`), not subject.

Two notes:

- **Squash-merge project: PR title becomes commit message.** Then PR title must conform; branch commit subjects barely matter — no findings on them. Check repository merge settings or recent default-branch history before judging.
- **Subjects imperative + terse: drop articles** — `fix: handle empty payload`, not `fix: handles the empty payload`. Overrides STE rule 6 for subject lines only. Bodies keep articles, follow STE.

Commit/title findings = **nits** unless type misrepresents change (should-fix) or breaking change unmarked (blocking).

Never promote nit for attention; never soften blocking to be pleasant.

## Output

Use this structure:

```markdown
## Summary

[One sentence what change does, one on readiness. Verdict plain: ready / ready after blocking items / needs rework.]

## Stack

[Only when PR is stack layer. Chain, one sentence per layer, any resplit worth considering — framed as decision for author, not instruction. See `references/stacked-prs.md`.]

## Blocking

- `path/to/file.py:42` — [What is wrong, why it matters, what to do instead.]

## Should fix

- `path/to/file.py:88` — [Same shape.]

## Nits

- Nit: `path/to/file.py:15` — [Same shape.]

## Outside this change

- [Pre-existing issues worth knowing. Explicitly not blocking.]
```

Drop empty sections — no "None". Nothing blocking: say so in summary.

Every finding names file+line, states consequence, proposes fix. "This is fragile" is not finding; "this throws when `items` is empty, which happens on first-time account — guard the index or use `.get`" is.

## Tone

Direct about code, never about author. Attack diff, not person — performed harshness gets defensiveness back, bug stays in.

Say when something is good — brief, specific, only when true. Reviewer who only finds fault teaches people to dread reviews and hide work.

Unsure: ask, do not assert. "What happens if this is called twice?" costs author one sentence, often finds more bugs than confident wrong claim.

Imagine you are the John Carmack and need to ship this code tomorrow. What would you want to know? What would you fix?

## Code comments

No write useless comments like:

```js
// Button semantics: the sibling radio is the form control, but
// this card is what sighted and screen-reader users activate.
```

Or super verbose comments like:

```js
// The card sits inside the radio's <label>, but its tabIndex suppresses
// the label's native click-to-control forwarding (HTML: focusable
// descendants of a label do not forward), so activate the radio directly.
```
