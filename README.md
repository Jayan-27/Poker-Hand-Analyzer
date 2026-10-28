# Hold'em Equity Lab

A Texas Hold'em equity calculator that runs entirely in the browser: no dependencies and no build step. You open `index.html` and it works.

## What it does

- **Exact equity** for heads-up spots on the flop, turn and river. It enumerates every live villain combo against every runout, which comes to about 1M showdowns on the flop in roughly 150 ms.
- **Monte Carlo equity** for preflop and multiway pots (up to 8 opponents). It reports a 95% confidence interval from the per-trial payoff variance, with split pots weighted 1/k.
- **Villain range modeling.** There is a 13×13 range grid, a top-X% slider and presets (UTG/CO/BTN opens, 3-bet range). The 169 starting-hand classes are ranked by simulated equity against a random hand, computed at load and cached.
- **Next-card equity map.** A 52-card heatmap shows hero equity after every possible turn or river card, with how far each card moves it.
- **Decision math**: pot odds, break-even equity, the EV of a call, and a fair-share comparison when checked to.
- **Showdown distribution**: how often hero finishes with each hand category.
- **Shareable state.** The hand, board and settings are encoded in the URL hash.

## Engineering notes

| Piece | Approach |
|---|---|
| Card model | Integers `0..51` (`rank*4 + suit`). Cards come from a single deck, so a card can't be selected twice. The picker disables used cards and the text parser rejects duplicates. |
| Evaluator | 13-bit rank masks per suit plus precomputed `POPCOUNT` and `STRAIGHT_HIGH` tables (8192 entries each). The result is one comparable 24-bit score (`category << 20` plus five 4-bit kickers), so comparing hands is just `>`. It makes no allocations in the hot loop and runs at about 2.5M 7-card evals/s on a laptop. |
| Correctness | 18 built-in self-tests: wheel and steel wheel, a straight and a flush that are *not* a straight flush, three pairs, two trips, kicker edge cases, board-plays chops. Equities match published values (AA vs KK 82.1%, AA vs random 85.2%, AKs vs QQ 45.8%). |
| Sampling | A seeded `mulberry32` PRNG makes runs reproducible. Each trial uses an "iteration stamp" array instead of reshuffling, so the per-trial reset is O(1). Rejection sampling draws range combos without conflicts. |
| Exact enumeration | Hero scores are cached per runout, which halves the evaluations on the flop. |
| Concurrency | The engine is a pure function that is serialized into a Blob-backed **Web Worker**, so the UI never blocks. Stale jobs are cancelled by terminating the worker. It falls back to the main thread if workers are unavailable. |

## Run

Open `index.html` in any modern browser. To host it, enable GitHub Pages on the repo.

The engine is plain JS between `// ENGINE-START` and `// ENGINE-END`, so it can also be loaded in Node for testing.
