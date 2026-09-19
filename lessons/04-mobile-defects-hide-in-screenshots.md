# 4. Mobile defects hide in screenshots

## What happened
We scored generated apps by looking at screenshots. Two of the three best-looking apps — one of them produced through the production route — **overflowed horizontally on a real 390 px viewport**. On the screenshots (taken at 500 px) they looked perfect.

Why 500 px: headless Chrome enforces a minimum window width of roughly 500 px, so "mobile" screenshots taken with `--window-size=390` are cropped or wider than they claim.

## How to measure at a true 390 px
Render the page inside an `<iframe width="390">` in a wrapper page, inject a small probe script into the app, and send the measurements to the wrapper with `postMessage`; the wrapper writes them into its DOM so `--dump-dom` returns them. Measure:
- horizontal overflow (`scrollWidth − innerWidth`) and the elements sticking out on the right (outside scrollable containers);
- tap targets smaller than 36 px;
- text smaller than 11 px;
- text contrast ratio (see caveat).

## What it found
Causes were **generic, not model-specific**:
1. a header with four navigation entries that does not fit in 390 px;
2. a grid child without `min-w-0`, forced to 427 px wide by long product names.

Both are the same classic CSS traps. A prompt rule ("no horizontal overflow") already existed and was too vague to help. We made it specific (`min-w-0`/`truncate` on flex/grid children, ≤ 3 items in a mobile header, navigation in a bottom bar). **We have not measured whether that helps** — one build proves nothing; the validator now reports overflow on every build so real data will tell.

Across **82 successful edits** measured before/after: none introduced overflow. The default cheap model did add more low-contrast text (12 across 13 edits) but **9 of those 12 came from a single edit**, so we read it as a possible hidden cost, not a systematic defect.

## Contrast is a signal, not a verdict
Our first version reported text-on-"white" ratios of 1.0 for white text on **gradient** backgrounds (the script assumed white when it found no solid background colour). Fix: if an element or ancestor has a background *image/gradient* or a translucent colour, mark the text unmeasurable and skip it. Even so, treat contrast numbers as approximate.

## Three separate scores
Keep **function** (does it work), **mobile** (objective measures) and **aesthetics** separate. A beautiful app can be broken; a correct one can overflow. Our aesthetics grid was 5 criteria × 0–2 (hierarchy, coherence, finish, local relevance, mobile usefulness of the first screen). It is **one grader, first screen only, 10 apps** — subjective by construction.

## Limits
Measured in headless Chrome, not on a physical phone. Soft-keyboard behaviour (especially iOS Safari) is untested.
