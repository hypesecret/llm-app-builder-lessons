# Lessons from building a prompt-to-app builder

Field notes and **real data** from building a product that turns a plain-language idea into a working web app with LLMs (single-file React apps, edits as SEARCH/REPLACE patches, models routed through OpenRouter). Written for people building something similar.

Everything here was measured on our own runs (≈ 100 model calls, ≈ $4 spent) and comes with its **limits**. Where we did not measure something, we say so instead of guessing.

## The short version

1. **Judge a model by cost per *successful* task, not cost per call.** A cheap model that fails often can cost more than an expensive reliable one. [→ 01](lessons/01-cost-per-successful-task.md)
2. **Don't pick one winning model.** Use a cheap model by default and a stronger one only after a *verified* failure. In our runs the priciest model was not the most reliable. [→ 02](lessons/02-model-routing-and-escalation.md)
3. **Validate before you show.** "It compiles" is not "it renders": one generated app compiled and displayed a blank page. A cheap compile check catches most failures; a headless-browser check catches the rest. [→ 03](lessons/03-validate-before-you-show.md)
4. **Mobile defects hide in screenshots.** Our two best-looking apps overflowed horizontally on a real 390 px viewport. [→ 04](lessons/04-mobile-defects-hide-in-screenshots.md)
5. **Prove your tests can fail.** Re-introduce the bug on purpose and watch the test go red. Ours had gaps we only found that way. [→ 05](lessons/05-testing-the-money-code.md)
6. **Most of our expensive mistakes were in the measuring, not the models.** [→ 06](lessons/06-mistakes-we-made.md)
7. **Cash is not capacity.** On a prepaid LLM balance, the constraint is the delay between a customer paying you and that money reaching the provider. One generic error for every system-side problem, one incident log, a worst case you can compute without statistics, and free trials refused first. [→ 07](lessons/07-cash-is-not-capacity.md) · package: [`llm-capacity`](https://github.com/hypesecret/llm-capacity)

## Headline numbers (edits, 15 requests × 8 models)

| Model | Attempts | Successes | $/attempt | **$/success** | s/attempt |
|---|---|---|---|---|---|
| deepseek/deepseek-v4.1-flash | 14 | 13 | 0.0044 | **0.0047** | 22 |
| openai/gpt-5.6-luna | 14 | 11 | 0.0039 | **0.0049** | 6 |
| deepseek/deepseek-v4-pro-0813 | 14 | 11 | 0.0097 | **0.0124** | 43 |
| z-ai/glm-5.3 | 14 | 10 | 0.0164 | **0.0230** | 9 |
| qwen/qwen3.8-max-0902 | 6 | 6 | 0.0246 | **0.0246** | 52 |
| google/gemini-3.8-flash | 6 | 4 | 0.0195 | **0.0292** | 16 |
| anthropic/claude-haiku-4.5 | 12 | 10 | 0.0249 | **0.0299** | 15 |
| anthropic/claude-sonnet-5 | 19 | 17 | 0.0586 | **0.0655** | 21 |

"Success" = the patch applied **and** the app compiled **and** it rendered a non-empty page in headless Chrome **and** the requested change was present. Sample sizes are small and unequal; differences of one or two attempts are noise. The raw rows are in [`data/`](data/).

## What this is not
- **Not a leaderboard.** One app, one prompt style, French-language requests, ≈ 100 calls, one week. Model names and prices are those of 2026-09-19 and will change. Use the *method*, re-measure on your own workload.
- **Not a claim about aesthetics.** We scored looks by eye on 10 apps, one grader; that is in lesson 04 and is explicitly subjective.
- The routing we ended up with is a starting point that we have **not** validated on production traffic.

## Reusable pieces extracted from this work
- [`search-replace-patch`](https://github.com/hypesecret/search-replace-patch) — tolerant applier for LLM SEARCH/REPLACE edits.
- [`openrouter-stream`](https://github.com/hypesecret/openrouter-stream) — streaming OpenRouter client with real-cost reporting and safe error classification.

## Layout
```
lessons/   six short write-ups
data/      raw benchmark rows (edits.json, builds.json) + schema
```

## Licence
MIT. Model outputs belong to their respective providers' terms.
