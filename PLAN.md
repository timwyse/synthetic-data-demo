# Judge the Judge — Workshop Plan

## Goal

An interactive workshop (~20 people, working in pairs → ~10 groups) on good
practices when evaluating synthetic data with LLM judges. Participants should
leave convinced — by their own hands-on experience — that known judge biases are
real, measurable, and that some can be fixed with better prompting while others
need pipeline-level fixes.

## The core idea

Instead of evaluating a dataset, we evaluate **the judges themselves**. We
measure any judge prompt on a fixed, pre-built evaluation set where the right
answers are known, and report a **bias dashboard**:

| metric | question it answers | fixable by prompting? (expected) |
|---|---|---|
| accuracy guardrail | is this judge competent at all? | — (must not regress) |
| surface appeal | does polish beat instruction-following? | largely yes |
| verbosity | does inflating a wrong answer rescue it? | partially |
| position | does presentation order decide verdicts? | no — fix is swap-and-aggregate |
| self-preference | is a wrong answer more convincing in the judge's own words? | no — fix is a different judge / panel |

The "fixable?" column is the workshop's punchline: participants discover it
empirically.

## Workshop flow

1. **Slideshow (presenter).** Set up the idea; show example pairs and let the
   room vote on which response is better before revealing the gold label and
   the judge's verdict. Cite the research (LLMBar/Zeng et al. ICLR 2024,
   Zheng et al. NeurIPS 2023, Panickssery et al. NeurIPS 2024, Wang et al. 2023,
   length-controlled AlpacaEval).
2. **Naive baseline, run live** (for the show — the *official* baseline is
   precomputed and cached so the leaderboard doesn't depend on the live run).
   Walk through the dashboard: every bias fires. Show the dose-response curve
   (accuracy vs padding length) and read the judge's own reasoning on pairs it
   got wrong.
3. **Participants' turn (in pairs).** Each pair writes a judge prompt and runs
   it. Goal: shrink the bias scores *without dropping the accuracy guardrail*
   (a judge that always answers "1" is perfectly consistent and perfectly
   useless). Iterate.
4. **Wrap-up.** Scorecard comparison: which biases moved, which survived every
   prompt. Lesson: your data-quality filter is only as good as your judge, and
   some judge failures can't be prompted away.

## The data

Built once by `prep_dataset.ipynb` from [LLMBar](https://github.com/princeton-nlp/LLMBar)
(instruction/response pairs with gold labels; adversarial pairs where the worse
response is superficially more appealing) plus generated probe variants:

- **control (20)** — untouched LLMBar Natural pairs (guardrail)
- **surface_appeal (24)** — untouched LLMBar Adversarial pairs (6 per subset)
- **verbosity (30 × 2)** — same Natural pairs, base vs wrong-answer padded to
  the dose a sweep finds the judge most fooled by (candidate doses
  0.7×/1.5×/2×/3×/5×, all pre-generated)
- **self-preference (30 × 2)** — same Natural pairs, wrong answer rewritten by
  the judge's own model vs by Claude (style transfer only; errors preserved)

Total: 164 pairs → 328 judge calls per run (every pair judged in both orders).
All generated artifacts are keyed by judge model so `gpt-4.1-nano` and
`gpt-4.1-mini` datasets coexist; the workshop notebook switches judges by
changing one constant.

## Infrastructure

- **Colab** notebooks; participants open a shared link (each gets their own
  runtime). Dataset fetched from this repo's raw URL on `main` (repo must be
  public, or participants clone with access).
- **One shared OpenAI API key**, spend-capped, shared in team chat and revoked
  after. Rate limits are the real constraint, not cost (a full run costs cents):
  harness caps concurrency at 5 per group and retries 429s with backoff.
- **Leaderboard**: to be decided — simplest is a final notebook cell POSTing
  metrics to a Google Apps Script webhook → Google Sheet projected at the front.

## Repository map

| file | purpose |
|---|---|
| `prep_dataset.ipynb` | run before the workshop: samples LLMBar, generates probe variants (with one Claude handoff), dose sweep, writes the dataset |
| `workshop.ipynb` | the day-of notebook: harness + bias dashboard + scorecard |
| `mvp.ipynb` | earlier two-round MVP (accuracy + position only); kept as fallback |
| `data/` (after prep) | datasets, generation caches, baselines — see PROGRESS.md |
| `PROGRESS.md` | current status and handover notes |
