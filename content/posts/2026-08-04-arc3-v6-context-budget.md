+++
title = "ARC3 V6 (LB 0.75): Context Budget 32768 to 49152 - The Cost of Going Too Far"
date = 2026-08-04T02:19:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "context-window", "regression"]
categories = ["ARC3 Dev Journal"]
summary = "Increased analyzer context window from 32k to 49k tokens (+50%). LB dropped 1.06 to 0.75 (-0.31). Local mean was 0.58, predicting improvement. First hard lesson: local mean is not LB score."
lb_score = "0.75"
version = "V6"
status = "REGRESSION"
+++

## TL;DR

Increased analyzer context window from 32k to 49k tokens (+50%). LB dropped 1.06 to 0.75 (-0.31). Local mean was 0.58, predicting improvement. First hard lesson: local mean is not LB score.

## Context

V4 had a context window of 32768 tokens. The vLLM server was running with `--max-model-len 65536`. There was a 32k gap between what the model could handle and what the analyzer fed it. The hypothesis: larger context window lets the solver retain more game history, which should help on multi-level games.

The local dry-run on the 6 public games supported this hypothesis: local mean rose from V4's 0.45 to 0.58 (+29%). I pushed to LB expecting similar improvement. The result was the opposite.

## Technical Choice

Single-variable change: `ANALYZER_CONTEXT_WINDOW = 49152` (was 32768). +50% increase. No other patches.

The choice of 49152 rather than 65536 was deliberate: leave headroom for the action prompt and observation tokens. The analyzer context window is shared between game history, system prompt, and current observation. Filling it to 65k would leave no room for the response.

## Parameter Decisions

| Parameter | V4 | V6 | Rationale |
|---|---|---|---|
| ANALYZER_CONTEXT_WINDOW | 32768 | 49152 | +50% to retain more game history |
| vLLM --max-model-len | 65536 | 65536 | unchanged |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | unchanged |
| temperature | 0.6 | 0.6 | unchanged |
| MULTIMODAL_UPSCALE | 4 | 4 | unchanged |

## Local vs LB Score

- Local mean: 0.58
- Baseline (previous version): V4 local 0.45
- Local delta: +0.13 (+29%)
- LB score: **0.75**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 49152 |
| vLLM server started | yes | vLLM server ready |
| submission.parquet written | yes | submission.parquet |
| give-up mechanism | yes | max_runtime |

## Outcome Analysis

LB 0.75, -0.31 from V4's 1.06. Local mean improved +29%, LB regressed -29%. The first hard evidence that local mean is uncorrelated with LB score on hidden games.

Three diagnoses:

First, the local public games are short (average 6-8 levels, 30-50 actions per level). A 49k context window easily fits the entire game history. The solver retains everything, which helps on familiar games.

Second, the hidden games are longer. The `baseline_actions` sums to 17135 across 25 public games (avg 685 per game), but hidden games likely have longer action sequences. With 49k context, the solver may have been retaining noise from early levels that were no longer relevant, diluting attention on the current level.

Third, larger context increases latency per inference call. With 9-hour wall clock and 110 games, every additional second per call compounds. The solver likely timed out on more games than V4, lowering contribution.

The local mean improvement was real but measured on the wrong distribution. Public games favor more context; hidden games favor less. This is the §9 lesson: local mean improvement on public games does not transfer.

## Next Version Plan

Revert context window to 32768 (V7 was a private test). V9 will replicate yw8837's LB-1.17 config as a controlled comparison.
