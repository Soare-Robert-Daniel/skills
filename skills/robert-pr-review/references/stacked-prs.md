# Reviewing a pull request inside a stack

Stack = chain of PRs where each targets branch below, bottom targets trunk. `gh stack` (GitHub official CLI extension, public preview mid-2026) one manager; Graphite, git-spice, Sapling, `git-town` others. Analysis below tool-agnostic.

Two changes when PR under review is stack layer. First, diff is deliberately incomplete — usual "this is unused" and "this has no tests" findings often wrong. Second, new review subject standalone PRs lack: whether split itself is good.

## 1. Read stack before diff

```bash
gh stack view --json          # full stack: branches, order, PR links
gh stack view --short         # branch names only
gh pr view <n> --json baseRefName,title,body,additions,deletions
gh pr diff <n>                # just this layer's diff
```

No `gh stack`, or other tool built it: reconstruct chain from base branches — PR whose base is not trunk sits on top of PR whose head is that base. Stack tooling usually writes checklist or ASCII chain into each PR body.

Record per layer: number, title, one-sentence purpose, size, current review state. Review state changes what advice is worth giving — see section 5.

## 2. Do not flag what later layer provides

Most common way stacked reviews waste author's time. Before any finding of form "X is unused", "X has no caller", "X is not tested", "X is never wired up", "this flag is never read": check layers above. Higher layer supplies it: drop finding.

Two exceptions:

- Layer above far away + intermediate layers ship public surface nothing uses yet. Exported function with no caller can get adopted by other teams before stack top lands.
- Dead interval large — hundreds of unreachable lines in trunk for days. Worth note even when intentional.

Review layer's own diff, not cumulative stack diff. `gh pr diff` and GitHub PR view scope correctly. Local diff: compare against layer's base branch, not trunk.

## 3. Two properties that decide whether split is correct

**Each layer independently reviewable.** Test: state layer's purpose in one sentence, no clause referring to another layer. Sentence needs "and", or needs "so that PR #103 can...": layer does more than one thing or cut at wrong seam.

**Every prefix of stack deployable.** Stacks merge bottom-up, one layer at a time. After layer 1 merges alone, trunk must stay correct and green. Then layers 1–2. Etc. Split that only works when whole stack lands together is not stack, it is one PR wearing three hats — someone merges bottom half on Friday: outage.

Prefix property = correctness bar, not taste. Violation is **blocking** on whichever layer breaks it. Everything else in this file advisory.

## 4. Split failure modes

Check against layer under review and neighbours.

**Ordering and prefix violations** (break trunk, so they block):

- Migration in higher layer than code reading new column. Bottom-up merge runs code first.
- Feature flag read in lower layer but defined in higher; or flag defaults on in lower layer while feature complete only at top.
- Lower layer imports from module a higher layer introduces.
- Tests for layer N's behaviour sitting in layer N+1. Layer N merges untested; trunk CI never covered it.

**Mixed concerns** (split real, cut wrong place):

- Refactor tangled with behaviour change in same layer. Reviewer cannot tell which line changed behaviour. Split refactor into layer below.
- One layer touches unrelated areas — auth, UI, docs — title needs "and".
- Formatting or lint-fix churn spread through functional layer instead of isolated in own layer.

**Churn across layers** (seam wrong, or layers should be one):

- Signature introduced in one layer, changed in next before anything calls it. Fold together; intermediate state has no reviewer value.
- Same function edited in three layers.
- Same helper added in two layers.
- Higher layer fixes bug lower layer introduced. Fold fix down — broken state never reaches trunk.

**Shape problems:**

- One 900-line layer + two 30-line layers. Cosmetic split; all review risk still concentrated in one place. Say plainly.
- Ten 40-line layers, arbitrary boundaries. Costs more to review than one coherent PR.

**Sync problems:**

- Lower layer changed after higher layers built, no restack. Higher layers reviewed against stale code. `gh stack sync` or `gh stack rebase` fixes; review waits until done.
- Local and remote stack composition diverged — chain on GitHub not chain author has.

## 5. Give author decision, not order

Author has context you lack — why seam is there, resplit cost, what is already approved. Present analysis and option; they choose. Concretely:

Show stack as you understood it, so wrong reading is easy to correct:

```markdown
## Stack

`main ← #101 ← #102 ← #103` — reviewing **#102**

| PR | Purpose in one sentence | Size | Merges alone? |
|----|------------------------|------|---------------|
| #101 | Extracts the token parser from `AuthService`. | 120 | yes |
| #102 | Adds refresh-token support to the parser. | 480 | yes |
| #103 | Adds tests for #101 and #102. | 90 | n/a |
```

State problem as consequence, propose one concrete alternative, name tradeoff:

> **#103 holds the tests for #101 and #102.** If #101 and #102 merge on their own — which is the point of stacking them — trunk gains a token parser and refresh-token handling with no coverage, and CI on trunk never exercises either. Suggest folding each test file down into the layer whose behaviour it tests, leaving #103 for the integration tests that genuinely need both.
>
> Cost: a restack and re-request of review on #102, which is already approved. Your call whether that is worth it here or worth doing differently next time.

Fix maps onto tooling: name operation. `gh stack modify`: `d` folds branch into one below, `u` folds into one above, `x` drops it, `Shift`+`↑`/`↓` reorders, `i`/`I` insert new layer. Then `gh stack submit` updates PRs. Preconditions: clean working tree, linear history, no rebase in progress, no merged or merge-queued layers — merged layer cannot be restructured.

**Review state governs advice.** Lower layer merged: resplit off table; say what you would have suggested, move on. Approved but unmerged: name cost of losing approval so author can weigh it. Nothing approved: resplit cheap, recommend without qualification.

## 6. Proportion

Stack-shape findings go in own `## Stack` section, above `## Blocking`, advisory — nit or should-fix — one exception: prefix violation that breaks trunk is blocking, goes in `## Blocking` with specific layer named.

Do not relitigate split on every layer. Say once, on layer where problem lives; reviewing several layers in one pass: shape analysis in one place, not repeated.

Split good: say so in one line. Splitting work well is real effort, invisible when it works.
