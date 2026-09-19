# 7. Cash is not capacity: running on a prepaid LLM balance

If you pay your LLM provider from a **prepaid balance** and your customers pay you **later, through slower rails** (mobile money, bank transfer), your real constraint is not the price of a token. It is the **delay between a customer paying you and that money becoming provider balance**. A wave of sign-ups can outrun the balance you hold at that instant.

What we changed, and what we could and could not verify.

## What we did

1. **One generic message for every system-side problem.** Provider "out of credit", rate limits, our own capacity gate: the user sees the same neutral sentence. Never "out of budget", never a provider name (a real 402 from OpenRouter contains a console URL with your key id, and we once shipped a screenshot with it). We do **not** write "our team has been notified" unless an alert channel is really configured and someone reads it: that would be a false promise.
2. **One incident log fed by two sources.** Server-detected errors and a user's "Report the problem" click land in the same table. The click sends **no cause**: the browser says "I have a problem", the server finds the technical context (model, escalation, duration). Repeated reports increment a counter; several users hit by the same code and model in a short window form a systemic incident.
3. **Capacity, not balance.** Compute exploitable balance (balance − absolute reserve − in-flight worst cases), autonomy in days (only with observed spend: never invent it), and the level left relative to the **recent peak** (the level right after the last top-up).
4. **A bounded worst case with no statistics.** If every account has a cost ceiling (subscribers per month, free trials per lifetime), the most you can still owe is a sum you can compute on day one. Compare it with the revenue those same accounts bring in: in our case the cash to front at the worst case is a small fraction of revenue, so the risk is dominated by the *delay*, not the amount.
5. **Refuse free trials first.** When cash is short, paying customers must not be starved by a wave of free accounts. First come, first served is the wrong policy.
6. **Fail open, opt in.** If the balance cannot be read, block nobody: a broken measurement must not stop the service. The admission gate is off unless explicitly enabled, because a wrong setting (a 10 $ reserve on an 8 $ balance) would silently block every free trial.

The reusable part is published as [`llm-capacity`](https://github.com/hypesecret/llm-capacity).

## Three traps we fell into

- **An ambiguous column in a SQL subquery gave a silently wrong number.** `where g.user_id = "user_id"` resolved to the inner table, so the condition was always true and every subscriber was charged with everyone's spend. The first attempt failed loudly (uuid vs text) and made us re-read the second, which would have produced a plausible wrong total with **no error**. Qualify every column in correlated subqueries, and test with two users who spend different amounts.
- **Integration-test files run in parallel by default and shared one database.** Each file emptied tables between tests, so files deleted each other's rows: five unexplained failures as soon as we added a second file. Run them sequentially when they share a database.
- **Write the incident before you refund the call.** Refunding deleted the log row the incident needed to read (model, escalation, duration).

## Limits

- The autonomy thresholds (7 days, 1 day, 30 % of peak) are starting values, not measured truths. Tune them to how long your top-up really takes.
- We verified the gate and the incident flow against a real provider outage (our key's credit really was exhausted: the user saw the generic message, an incident was logged, nothing was charged). We did **not** verify the balance endpoint against a live account, only its documented response shape.
- No alert channel was configured when this was written.
