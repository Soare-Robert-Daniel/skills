# Simplified Technical English for comments and commit messages

ASD-STE100 = controlled-language spec for aerospace maintenance manuals. Full spec: ~65 writing rules + dictionary of approved words. This file: rules that carry over to code comments, plus how to apply.

Point is not compliance theatre. Comments get read by tired people, non-native English speakers, future maintainers with no context. Short declarative active-voice sentences survive that; dense prose does not.

When flagging comment, quote rule number below. Vague "STE style" feedback does not tell author what to change.

## Rules that matter for comments

**1. One idea per sentence.** Split sentence carrying two facts. Aim ≤20 words.

**2. Active voice.** Name actor.
- No: `// The cache is invalidated when the record is updated.`
- Yes: `// updateRecord invalidates the cache.`

**3. Simple tenses only.** Simple present, simple past, simple future. No perfect or progressive tenses.
- No: `// We have been retrying here because the API had been failing.`
- Yes: `// The API returns 503 under load, so this retries three times.`

**4. One word, one meaning.** Pick term, keep it. Code says `user`: comment says `user` — not `account`, `member`, `customer`. Inconsistent vocabulary merges two concepts in reader's head.

**5. No -ing phrases as modifiers.** Leading present participle forces reader to hold clause open.
- No: `// Locking the mutex, we then write the buffer.`
- Yes: `// Lock the mutex. Then write the buffer.`
- Participles fine inside established technical names (`floating point`, `string formatting`).

**6. Keep the articles.** Telegraphic comments read ambiguous.
- No: `// Set flag if buffer full.`
- Yes: `// Sets the flag if the buffer is full.`
- Exception: Conventional Commits subject lines are imperative and length-capped, drop articles — `fix: handle empty payload`. Commit bodies keep them.

**7. Noun clusters: three words maximum.** Break longer with prepositions.
- No: `// legacy user session token cache key`
- Yes: `// the cache key for a token from a legacy session`

**8. No idioms, slang, figures of speech.** Do not survive translation, often not six months.
- No: `// Belt and braces — bail out early if things look dicey.`
- Yes: `// Returns early if the payload fails validation. The caller also validates.`

**9. Define abbreviation on first use in file,** unless standard in codebase. `TTL` fine. `mrgd_evt_bfr` not.

**10. "Must" = requirement, "can" = possibility.** Avoid "should" — reader cannot tell rule from suggestion.
- No: `// Callers should hold the lock.`
- Yes: `// The caller must hold the lock before it calls this.`

**11. Warnings come before the thing they warn about,** start with instruction.
- No: `// This deletes all rows if you pass an empty filter, be careful.`
- Yes: `// Do not pass an empty filter. An empty filter deletes all rows.`

## Applying in review

Say what rule comment breaks, offer rewrite. Do:

> Nit: `sync.py:31` — Passive and two ideas in one sentence (STE 1, 2). Suggest: `// syncWorker writes the manifest. The upload step reads it.`

Not:

> This comment isn't STE compliant.

## Scope and proportion

STE applies to comments, docstrings, commit messages, PR description.

Not identifiers, string literals, user-facing copy, test names — `test_returns_empty_when_no_rows_match` does its job; do not reword for sentence length.

STE findings almost always **nits**. Two exceptions rise higher: comment contradicting code, and warning comment ambiguous about whether mandatory. Correctness problems wearing style costume.

Badly worded comment = smaller problem than missing one. Do not spend review polishing prose in file with no explanation of why any of it exists.

Check whether comment should exist at all before applying any rule here. Comment narrating change or addressing reviewer: delete, not rewrite — see "Comments" in SKILL.md. Rewriting `// Added this to fix the bug` into clean STE produces well-formed sentence that still does not belong in file.
