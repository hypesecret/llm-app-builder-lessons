# 1. Cost per successful task, not cost per call

## The metric
```
cost per success = (cost of ALL attempts, failures included) ÷ (number of successes)
```
A model at $0.02 per call that fails 40 % of the time costs about $0.033 per success. A model at $0.03 that always works costs $0.03. Average cost per call hides this completely.

Define "success" strictly, or the number is flattering. Ours required all four:
1. the patch applied,
2. the result compiled,
3. the app **rendered** something (headless Chrome, non-empty root),
4. the requested change was actually present.

## What we measured
Edits to one single-file React app, 15 requests across three difficulty tiers (change a price; add an isolated section; add real logic). See the table in the [README](../README.md) and [`data/edits.json`](../data/edits.json).

- DeepSeek v4.1 flash: **$0.0047 per success** (13/14). Claude Sonnet 5: **$0.0655** (17/19). About **14× apart**, with reliability that is not significantly different on this sample.
- Most model families failed about **1 time in 5 on the largest edits** (two models did not, but on very small samples: 4 and 2 attempts on that tier). In our sample the cheap models did not fail visibly *more* often on small edits; the failures concentrated on the big ones. With 4–5 attempts per model and tier, that is an impression, not a result.

## Input dominates edit cost on a single-file app
Changing one price cost **$0.047** on the priciest model, for **144 output tokens**. The bill was almost entirely input: ~21 000 tokens = ~6 900 of system prompt + ~14 000 of the app's own code, re-sent on every edit. So one small edit cost about **30 % of a full build**.

Levers, in the order they paid off:
1. **A cheaper model** for small edits (10–15× per success).
2. **Prompt caching** of the system prompt: −33 % on that model (0.0469 → 0.0313). Limits: it lasts minutes, and it cannot cache the app code (which changes every version).
3. (Not done) sending less code — needs an index of the code, which a single file does not have.

For a **full build** it is the opposite: ~95 % of the cost is output tokens.

## Traps
- **A case is not a verdict.** The same model on the same request succeeded once and failed once. Replay cases, and count *every* run. Our first evaluation kept only the first run of a replayed case and hid this.
- **Reasoning tokens are output tokens.** One model spent its entire 16 000-token budget "thinking" and returned nothing, for $0.17. Cap reasoning per task.
- **Report cost as `null`, never `0`, when the provider gives none.** An invented zero makes a broken run look free.
- **Fix the sample size before running.** We spent $3.22 on ≈ 118 calls with no stopping rule set in advance; the person paying judged the data sufficient and had us stop and finish the analysis offline on the stored outputs. Decide beforehand how many runs are enough, and keep raw outputs so analysis never needs new spend.

## Limits
One app, ≈ 100 calls, 8 models, prices of 2026-09-19. Unequal samples (6 to 19 attempts per model). Treat rankings as "who to test first", not as a result.
