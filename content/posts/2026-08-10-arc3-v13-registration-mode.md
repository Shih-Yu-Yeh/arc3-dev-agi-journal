+++
title = "ARC3 V13 (LB 0.87): Tanaka Safety v1 + Registration Mode - Skipping the 9-Hour Commit"
date = 2026-08-10T12:48:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "registration-mode", "pipeline"]
categories = ["ARC3 Dev Journal"]
summary = "Used a 1-row placeholder submission.parquet to skip the 9-hour commit run. The hidden rerun produced the real LB 0.87. Pipeline trick, not a solver improvement."
lb_score = "0.87"
version = "V13"
status = "PIPELINE_TRICK"
+++

## TL;DR

Used a 1-row placeholder submission.parquet to skip the 9-hour commit run. The hidden rerun produced the real LB 0.87. Pipeline trick, not a solver improvement.

## Context

Each full commit run takes 9 hours on an RTX Pro 6000. With 30 hours of weekly GPU quota, that limits me to 3 full submissions per week. The bottleneck is not the solver; it is the commit run that plays the 25 public games.

V13 tested a pipeline trick: write a 1-row placeholder submission.parquet during the commit run, skip the actual game playing, and let Kaggle's hidden rerun produce the real score. The trick is documented in `tanakaai24/arc3-qwen3-6-duck-lb117-safety-v1`.

## Technical Choice

The notebook detects `KAGGLE_IS_COMPETITION_RERUN` env var. If False (commit run), it writes a placeholder:

```python
submission = pd.DataFrame(
    data=[["1_0", "1", True, 1]],
    columns=["row_id", "game_id", "end_of_game", "score"],
)
submission.to_parquet("/kaggle/working/submission.parquet", index=False)
```

If True (hidden rerun), it runs the full TAAF solver. The commit run completes in 2 minutes (vs 9 hours), saving 8h 58min of GPU quota per submission.

## Parameter Decisions

| Parameter | V10 | V13 | Rationale |
|---|---|---|---|
| commit run duration | 9 hours | 2 minutes | Skip game playing |
| hidden rerun | normal | normal | Unchanged |
| submission.parquet (commit) | real scores | 1-row placeholder | Registration only |
| solver | stock TAAF | stock TAAF (tanaka safety v1) | Same as V10 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | unchanged |

## Local vs LB Score

- Local mean: n/a (commit run skipped)
- Baseline (previous version): V10 LB 0.79
- Local delta: n/a
- LB score: **0.87**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| registration mode | yes | Wrote Kaggle code-submission registration artifact |
| submission.parquet written | yes | submission.parquet |
| taaf_setup_env.json | yes | taaf_setup_env.json present |
| (solver patches) | n/a | Hidden rerun runs the real solver |

## Outcome Analysis

LB 0.87, +0.08 over V10's 0.79. Within hidden rerun variance, so effectively equivalent to V10.

The registration-mode trick worked. The commit run completed in 2 minutes. The hidden rerun ran the real TAAF solver and produced LB 0.87.

This is a pipeline trick, not a solver improvement. The LB 0.87 is the tanaka safety v1 fork's true hidden-game score, which is +0.08 above V10's pure baseline.

The implication for resource planning: with this trick, I can submit 3-5 versions per day (limited by hidden rerun queue, not commit run duration). The bottleneck shifts from GPU quota to Kaggle's hidden rerun queue depth.

Caveat: the trick assumes the hidden rerun actually runs the solver. If the notebook writes a placeholder but the hidden rerun also writes a placeholder (e.g. due to a setup failure), the LB returns 0.00. This is exactly what happened to V21.

## Next Version Plan

V14 will test temperature 0.3 (down from 0.6). The hypothesis: lower temperature produces more deterministic, reproducible actions, which should help on hidden games with strict win conditions.
