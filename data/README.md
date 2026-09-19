# Data

Raw rows behind the tables. No prompts, no keys, no personal data.

## `edits.json` — 99 rows
One row per edit attempt (a request sent to a model on the same base app).

| Field | Meaning |
|---|---|
| `case` | request id: `m-*` small change, `l-*` isolated addition, `f-*` real logic |
| `category` | `micro` / `local` / `feature` |
| `model` | OpenRouter model id |
| `costUsd` | cost billed by OpenRouter for that attempt |
| `inputTokens`, `outputTokens`, `cachedTokens` | real usage |
| `seconds` | duration |
| `success` | patch applied **and** compiled **and** rendered in headless Chrome **and** the requested change present |
| `failure` | why it failed, when it did |

Some cases were run more than once for the same model; **every run is a row** (that is how the variability shows).

## `builds.json` — 13 rows
One row per full build (a new app from one prompt): cost, duration, output tokens, output tokens/second, whether it was complete, compiled, and **rendered in Chrome**.

## `_tables.md`
The two tables in the README and lesson 02, generated from these files.

## Caveats
- Sample sizes are small and unequal. One base app, French-language requests.
- Prices and model ids are those of 2026-09-19.
- "Renders" means the mount point contained text in headless Chrome after load; it is not a functional test of every button.
