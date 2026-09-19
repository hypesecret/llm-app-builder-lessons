# 3. Validate before you show

The question a server-side validator must answer is small: **"can this be shown to the user without being obviously broken?"** It does not judge beauty.

## Levels, cheapest first
| Level | Check | Cost |
|---|---|---|
| patch | the model's edit blocks apply to the current code | ms |
| structure | full document, has a mount point (`#root`) | ms |
| compile | the JSX/TS compiles (esbuild) | ms |
| render | headless Chrome loads it and the root has text | 2–12 s, needs a browser |
| runtime | JavaScript errors at load (warning only) | with render |
| quality | warnings: no viewport meta, horizontal overflow at 390 px… | with render |

Verdict: **`PASS` / `PASS_WITH_WARNINGS` / `FAIL`. Only `FAIL` triggers an escalation** — a warning must never cost you a second model call.

## "Compiles" is not "renders"
One generated app compiled fine and displayed a **blank page** (a runtime error). Without the render level it would have shipped. The render check was the deciding factor in our real escalation test.

## The cheap filter and the real judge disagree sometimes
We used esbuild as a fast compile filter and the browser's own Babel (in the generated page) as the real runtime. They are not equivalent, and **the esbuild version changes verdicts**: after a dependency bump one app flagged "invalid JSX" compiled. We re-judged all 7 flagged cases in Chrome: **6 were truly blank pages, 1 rendered fine.** Treat the compiler as a filter and the browser as the judge.

## How we tested the validator without spending money
Re-run it over stored outputs and compare with an independent "ground truth" (the browser):
- 14 stored builds → 14/14 verdicts equal to the browser's;
- 89 stored edits → every one of the 13 broken patches detected, 0 false alarms.

## Design notes
- Do the check **server-side and before streaming the final answer**, but you can still stream the first attempt to the client live: if validation fails and you escalate, emit a marker (`<!-- RESTART -->`) that tells the client to discard everything before it.
- Truncated output is a distinct failure: do not escalate it.
- If the browser probe returns nothing (page hung), **do not judge** rather than inventing a verdict.
- **Security:** you are executing model-generated code on your server. Run the browser in a container **with no access to your internal network**. A host allow-list (`--host-resolver-rules`) limits names but **does not cover literal IP addresses**.
- Run the validator on the cheap levels always, on the browser level only where it pays (new builds and large edits), because it adds seconds.

## Limits
The browser level was proven on real failures in two runs (one edit, one build) and offline on stored data — not at volume. It requires a server with Chrome; it cannot run on typical serverless platforms.
