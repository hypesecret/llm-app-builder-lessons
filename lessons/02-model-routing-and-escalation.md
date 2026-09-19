# 2. Routing: cheap by default, escalate on a verified failure

## The principle
Do not look for the best model. Use **several specialised models and escalate only when needed**:

```
cheap model → generate → validate ─ PASS ─→ show
                            └ FAIL ─→ stronger model (ONE attempt) → validate → show / controlled error
```
The expensive model is the exception, not the default. What makes this safe is the validator (lesson 03): without a reliable "did it work?" signal you cannot escalate on evidence.

## Building a new app is a different job from editing one
Full builds, one prompt each, two apps (a booking app and a shop):

| App | Model | Cost | Time | Output tok/s | Result |
|---|---|---|---|---|---|
| salon | anthropic/claude-sonnet-5 | $0.216 | 146 s | 136 | ✅ renders |
| salon | deepseek/deepseek-v4-pro-0813 | $0.032 | 442 s | 22 | ✅ renders |
| salon | z-ai/glm-5.3 | $0.039 | 33 s | 246 | ❌ compiles but blank page |
| salon | qwen/qwen3.8-max-0902 | $0.067 | 198 s | 55 | ✅ renders |
| salon | google/gemini-3.8-flash | $0.048 | 60 s | 203 | ✅ renders |
| salon | openai/gpt-5.6-luna | $0.010 | 35 s | 225 | ✅ renders |
| salon | openai/gpt-5.6-sol | $0.128 | 114 s | 105 | ✅ renders |
| boutique | anthropic/claude-sonnet-5 | $0.257 | 170 s | 141 | ❌ truncated by the output cap |
| boutique | deepseek/deepseek-v4-pro-0813 | $0.060 | 432 s | 47 | ❌ does not compile |
| boutique | z-ai/glm-5.3 | $0.050 | 39 s | 337 | ❌ does not compile |
| boutique | qwen/qwen3.8-max-0902 | $0.071 | 208 s | 56 | ✅ renders |
| boutique | google/gemini-3.8-flash | $0.069 | 84 s | 211 | ✅ renders |
| boutique | openai/gpt-5.6-luna | $0.013 | 61 s | 183 | ❌ does not compile |

(The truncated Sonnet build is *our* fault: we had lowered the output cap to 24 000 tokens without measuring. See lesson 06.)

- **Speed is a selection criterion, not a footnote.** One model built the salon for $0.032 — but in **442 s**, past our 300-second request limit. Cheap and good is useless if it is too slow to serve.
- Best trade-off for building in this sample: a mid-price fast model that rendered 2/2 with the best-looking result (by our subjective scoring, lesson 04). Two builds per model is far too few to be confident; it is where we *started*.
- Different models were best at building and at editing. Keep two tables.

## What escalation really costs
We forced a cheap model to fail on a real build and let the system escalate: the cheap attempt produced a page that compiled but rendered blank (caught by the headless-browser check), the strong model then succeeded. **Total: $0.32 and 236 s** for that one build — roughly five times a successful cheap build, and close to the request limit. Escalation is a safety net, not a free lunch.

Untested ideas that follow from this: retry the *same* cheap model first (its success varied run to run, for ~$0.05), or feed the validator's error text to a cheap model for a targeted repair instead of regenerating. Both are hypotheses.

## Rules we ended up with
- One escalation, never a loop.
- **No escalation on truncated output** — a stronger model will truncate too.
- Check budget and remaining time before escalating (a build plus an escalated build can exceed the request limit).
- The escalation tier must differ from the first attempt's model.
- Give the router a fallback *list* per task so a rate-limited provider does not stop the request.
- Some endpoints **require reasoning** and reject "reasoning disabled": handle that error by retrying with minimal reasoning ([`openrouter-stream`](https://github.com/hypesecret/openrouter-stream)).

## Limits
Two apps per model for builds; 14–19 edit attempts per model. Nothing here is validated on production traffic; log the model, category, cost, validation verdict and escalation flag for every call so you can re-derive this table from real usage.
