# 5. Testing the code that guards your money

If your product spends real money per request, the quota, budget and cost-tracking logic is the most valuable code to test — and the easiest to test badly.

## What we tested, and where
- **Pure logic** (token caps, cost estimates, env overrides, plan parsing, patch application, error mapping): unit tests.
- **The database-backed guards** (free-prompt quota, per-device and per-IP caps, daily budget, per-user budget, concurrency, "reserve the worst case of in-flight calls", escalation excluding itself, cumulative cost, refund of a failed call): **integration tests on a real Postgres** (24 tests). Mocking the database would have tested our mocks.
- **The provider client** with a simulated `fetch` and no network.
- **The validator** on real stored outputs, both good and broken.

135 tests in total, run in CI on every push, including migrations applied to a fresh Postgres and a production build with **no secrets**.

## A test that has never failed proves nothing
All 135 passed on the first run, which is suspicious. So for each important protection we **re-introduced the real bug on purpose** and checked that a test went red, then restored the code:

| Re-introduced defect | Caught? |
|---|---|
| build output cap back to the value that truncated a real app | yes |
| provider error message returned to the user verbatim (leaks key id) | yes (6 tests) |
| retry on "reasoning mandatory" removed | yes |
| worst-case reservation of in-flight calls removed | yes |
| concurrency cap removed | yes |
| refund no longer deletes the row | yes |
| ambiguity refused only on the *exact* path of the patch matcher, accepted on the tolerant path | **no → gap** |
| HTTP 402 not recognised as "credits exhausted" (message text alone was enough) | **no → gap** |

The last two were **real coverage gaps**, found only this way, and fixed by adding one test each.

## Traps in the method itself
- **Check the mutation actually applied.** Twice, our `sed` pattern did not match and the "mutation" changed nothing, so the tests "passed" and we nearly concluded they were fine. Compare the file before/after (`cmp`, `git diff`) and fail loudly if unchanged.
- **A CI workflow that has never run proves nothing.** Ours failed on its first run: a peer-dependency conflict that `npm install` tolerated locally and `npm ci` (strict) rejected. Simulate the strict command (`npm ci --dry-run`) before pushing, and watch the first run.
- Also assert **the principle**, not just the value: e.g. "the escalation tier's first model differs from the cheap tier's", rather than pinning a model name that will change next month.

## Limits
Not covered: the browser-render level (needs Chrome, verified by scripts), the UI, the full HTTP routes with streaming, authentication, real email sending, load.
