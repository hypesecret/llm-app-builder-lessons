# 6. Mistakes we made (so you can skip them)

Most of the costly ones were in our **measuring**, not in the models.

## Measuring
- **We published a report with wrong data.** We re-ran an evaluation with a *relative* path in a `file:///../x.html` URL. That URL is invalid, the browser loaded nothing, and every good app was marked "blank page". We had only looked at the top of the report. → Use absolute paths; a check that fails silently is worse than one that crashes; **read the whole report before handing it over**; when every attempt of a model "fails" for the same reason, suspect the checker first.
- **Our own checker produced false failures — twice.** A regex demanded `<script type="text/babel">` exactly; one model wrote `<script type="text/babel" data-presets="react">` and both its apps were marked "no script". Another looked for a word ("testimonial") in code where the model had used a different heading. → Always keep raw outputs so you can **re-evaluate offline** without paying again.
- **Our de-duplication hid real variability.** A case run twice was counted once, keeping the failing run. Counting all runs showed the same model succeeding once and failing once.

## Limits and errors we introduced
- **We lowered an output cap without measuring** (32 000 → 24 000 tokens) and a bigger-than-the-first-one app was truncated: $0.257 for an unusable result. A cap that is too low costs more than one that is too high. Never tighten a limit from one data point.
- **We treated every HTTP 503 as "engine not configured"** in the client. A new server-side 503 (daily budget reached) would have told users their API key was missing.

## Errors shown to users
- **A provider error message was sent verbatim to the browser** and contained a console URL with the API key's identifier. → Map errors to your own messages; log the detail server-side; alert the operator when it means "credits exhausted".
- **In the workspace view an engine error was invisible**: the user saw their own message and then nothing. The error text was only rendered on the landing view. A test that *simulates failures* finds this; the happy path never will.
- **A screenshot of that error page was committed to the repository** and showed the key identifier. Review screenshots before committing them, like you review text.

## Process and tooling
- **Killing processes by image name** (`taskkill /IM node.exe`) targets everything on the machine. Kill the process that owns the port, or a known PID. A filter on *command line* also matched the parent shells that launched the command.
- **Long shell heredocs with quotes and apostrophes** failed to parse (`unexpected EOF`) and applied nothing — three times. Write files with a file tool, not a nested shell command.
- **Files shared between two runtimes**: a file written to `/tmp` by a POSIX-style shell was not where the Node process (native Windows) looked. Use one explicit absolute path.
- **Background jobs started with `&`** died when the launching shell exited; run them as tracked background tasks.
- **A dev-server "SyntaxError: Invalid or unexpected token"** appeared once, on a cold start, and never again in four runs. We could not identify the cause; we recorded it instead of inventing one.

## Decisions we changed our mind about
- We planned a "complexity router" (scores, weights, RouteLLM, LiteLLM) before having data. With real data, **simple rules + logged costs** were enough, and the routing that mattered was *which model for which job*.
- We assumed a single component library would fit; it turned out our runtime (one HTML file, no bundler) could not run it — a survey of what exists, **done before coding**, would have said so in an hour.
