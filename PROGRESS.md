# Progress & Handover

Status as of 2026-09-04. See PLAN.md for the design; this file tracks where we
are and what happens next, so anyone (including a fresh Claude session) can pick
up mid-stream.

## Done

- [x] Workshop design settled: bias dashboard (guardrail, surface appeal,
      position, verbosity, self-preference); presenter runs the naive baseline,
      participants (in pairs) write informed prompts and see which biases move
- [x] `mvp.ipynb` — early two-round harness, validated on Colab with
      gpt-4.1-nano (finding: naive ≈ rules at small n; motivated bigger samples
      and the mini A/B)
- [x] `workshop.ipynb` — day-of notebook: double-order harness, bias dashboard,
      scorecard, misjudged-pair exhibits; degrades gracefully if probe
      categories are missing from the dataset
- [x] `prep_dataset.ipynb` — sampling, judge-model rewrites (cached per model),
      Claude task export, dose sweep, final dataset build; all judge-dependent
      artifacts keyed by model so nano/mini coexist
- [x] Merged to `main` (PR #1)

## Decided

- Sample sizes: `N_CONTROL = 20`, `N_SURFACE_PER_SUBSET = 6`,
  `N_VERBOSITY = 30`, `N_SELFPREF = 30` (164 pairs, 328 calls/run). Bigger
  full-dataset baselines for slide numbers deferred — additive later if needed.
- Participants work in **pairs** (~10 groups) — halves rate-limit burst.
- Judge model: start with `gpt-4.1-nano`; build the `gpt-4.1-mini` dataset too
  and A/B before the day. Prompting gains concentrate in stronger judges, so if
  nano shows no round-2 lift, mini is the workshop judge.
- Naive baseline: **precomputed and cached** as the official leaderboard
  baseline; also run live during the talk for the show (they should match).
- Verbosity padded dose: chosen empirically by the prep dose sweep (the dose
  the naive judge is most fooled by); the sweep curve doubles as a slide.

## In flight

- [ ] **(presenter, local)** Run `prep_dataset.ipynb` top → handoff marker with
      the constants above, once per judge model (identical constants both
      runs). NOTE: if a `data/own_rewrites_*.json` exists from an earlier run
      with different constants, delete it first — the cache is positional.
      Then commit + push `data/` to `main`.
- [ ] **(Claude session)** On seeing `data/to_generate.json` on `main`:
      generate `data/claude_generations.json` (150 paddings across 5 doses +
      30 rewrites; style transfer only, errors preserved, `generator_model`
      stamped), plus in the same PR: harden the rewrite cache (store question
      identity, fail loudly on count/seed mismatch) and add the
      precomputed-baseline cells (prep saves `data/baseline_<model>.json`;
      workshop loads it as the official `naive` run). A check-in trigger is
      armed to catch the push automatically.
- [ ] **(presenter, after merging that PR)** Pull, run the prep "after the
      handoff" section per model: dose sweep → `dose_curve_<model>.png` +
      final `data/workshop_pairs_<model>.json`. Spot-check the QA cells
      (variants must preserve the original's errors). Commit + push.
- [ ] **(presenter)** A/B the judges: run `workshop.ipynb` naive vs an informed
      prompt under both models; pick the workshop judge.

## Still to build / decide

- [ ] Leaderboard mechanism (proposal: Apps Script webhook → Google Sheet;
      fallback: groups read scores aloud)
- [ ] Slides (theory section, room-vote examples, dose curve, expected-outcomes
      table)
- [ ] Load test: 2–3 simulated groups running concurrently against the real key
      (checks tier/RPM); check the key's usage tier well before the day
- [ ] Make repo public (workshop notebook fetches the dataset via raw URL) —
      or decide participants clone with access
- [ ] Spend cap on the OpenAI project; plan to rotate the key after the session

## Handoff protocol (Claude generations)

1. Presenter pushes `data/to_generate.json` (+ partial datasets) to `main`.
2. Claude generates `data/claude_generations.json`:
   `{"generator_model": ..., "generations": [{"pair_id", "task",
   "dose" (pad only), "text"}]}` — pad tasks exist at every dose per question;
   rewrite tasks once per question.
3. Claude opens a PR to `main`; presenter reviews the QA-relevant bits (do
   variants preserve the flaws?) and merges.
4. Presenter continues with the after-the-handoff prep section.

## Key constraints to remember

- Colab secrets don't travel with a shared notebook — distribute the key via
  team chat, spend-capped, revoke after.
- OpenAI rate limits are per organization per model, not per key; multiple keys
  don't help. Tier upgrades need spend + elapsed time — check a week ahead.
- Don't change sampling constants or SEED between the nano and mini prep runs,
  or after the handoff — the generated tasks are tied to the sampled questions.
- `workshop.ipynb` asserts the dataset's `judge_model` matches its `MODEL`
  constant; self-preference is only valid for the model it was built with.
