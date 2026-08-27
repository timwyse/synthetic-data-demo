# Synthetic Data Evaluation Workshop — Plan

## Goal

An interactive workshop on good practices when evaluating synthetic data (i.e. judging the quality of data you've generated to fine-tune a model with). Participants should leave convinced — by their own hands-on experience — that known LLM-judge pitfalls are real, and that applying the research-backed mitigations makes their evaluations measurably more reliable.

## The core idea

Instead of evaluating a dataset, we evaluate **the judges themselves**. Participants write their own "LLM-as-judge" prompts, and we score those prompts against a benchmark where the right answers are known in advance. That gives us a leaderboard, and a before/after comparison.

## Workshop flow

1. **Round 1 — naive judging (before the theory).**
   Participants get a set of response pairs and 5 minutes to write a prompt that picks the better response in each pair. Everyone's judge is scored automatically. Most will do worse than they expect — the pairs are designed to trip up naive judges.

2. **Theory — why did our judges fail?**
   A short research-backed tour of the known pitfalls of LLM judges:
   - Self-preference: models favor their own generations
   - Verbosity bias: longer answers score higher, regardless of quality
   - Position bias: the order of two answers changes the verdict
   - Surface appeal: confident, well-formatted answers beat correct ones
   - Plus mitigations from the literature: rubrics, position swapping, verify-then-judge, judge panels, length control

3. **Round 2 — informed judging.**
   Participants rewrite their judge prompts applying what they've learned. Judges are re-scored. The leaderboard shows everyone's Round 1 vs Round 2 accuracy — the improvement is the payoff.

4. **Wrap-up.**
   What this means for real synthetic-data pipelines: your data filter is only as good as your judge.

## The data

We use **LLMBar** (Princeton NLP, ICLR 2024): ~400 instruction/response pairs with gold labels for which response is objectively better. Crucially, its "Adversarial" subsets were built so the *worse* answer looks superficially more appealing — exactly the traps from the theory section. Even GPT-4 as a naive judge scores near chance on them. We'll sample a few dozen pairs, plus a handful of custom pairs covering verbosity and self-preference specifically.

## What we measure

For each submitted judge prompt:

- **Accuracy** against the gold labels (overall and per trap category)
- **Position-flip rate**: how often the verdict changes when we swap the order of the two answers
- **Verbosity tendency**: how often the judge picks the longer answer when it's the wrong one

## What we need to build

- A small app (or notebook) where participants paste in their judge prompt and hit run
- A scoring harness that runs the prompt over the pair set (in both orders) and computes the metrics above
- A live leaderboard to project during the session

## Open questions for collaborators

- Which model API(s) will the workshop use, and do we have keys/budget for ~100 judge calls per participant?
- How many participants, and how long is the session?
- Do participants run things on their own laptops (needs a hosted app) or do we run submissions from the front of the room (low-tech, zero setup)?
- Which domain should the custom pairs come from — generic instructions, or something closer to our team's actual synthetic data?
