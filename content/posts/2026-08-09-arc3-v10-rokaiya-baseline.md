+++
title = "ARC3 V10 (LB 0.79): Rokaiya Pure Baseline - Deliberate Regression for Calibration"
date = 2026-08-09T01:09:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-6", "baseline", "calibration"]
categories = ["ARC3 Dev Journal"]
summary = "Deliberately stripped all patches, grafts, and prompt addenda. LB 0.79. Calibration baseline to measure the absolute floor of the TAAF harness."
lb_score = "0.79"
version = "V10"
status = "CALIBRATION"
+++

## TL;DR

Deliberately stripped all patches, grafts, and prompt addenda. LB 0.79. Calibration baseline to measure the absolute floor of the TAAF harness.

## Context

After V9's controlled comparison, I needed to know the absolute floor. What does the Tufa harness score with zero patches?

V10 deliberately removed: AGI_8 patch, AGI_9 patch, program synthesis prompt, and any grafts. Just the stock TAAF bundle with Qwen3.6-27B-FP8.

The point was not to improve LB, but to establish a calibration point. Every future patch's contribution could then be measured against this floor.

## Technical Choice

Used `rokaiyasomapti/arc3-duck-v12-1d7d88-27e1af` as the reference. This kernel was a pure baseline replica without modifications. I forked it to verify that the LB 1.38 the original claimed was reproducible.

The choice of rokaiya's kernel rather than V2 was deliberate: V2 had the Tufa bundle, but rokaiya's was a cleaner baseline with fewer modifications. If both scored similarly, the floor was confirmed.

## Parameter Decisions

| Parameter | V9 | V10 | Rationale |
|---|---|---|---|
| AGI_8 patch | on | off | Strip all patches |
| AGI_9 patch | on | off | Strip all patches |
| program synthesis prompt | off | off | Already off in V9 |
| grafts | none | none | Stock |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | unchanged |
| context window | 32768 | 32768 | Stock |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V9 LB 1.10
- Local delta: n/a
- LB score: **0.79**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## Outcome Analysis

LB 0.79. The calibration floor.

The rokaiya kernel was supposed to score 1.38, but my fork scored 0.79. The 0.59 gap is attributable to two factors:

First, rokaiya's claimed 1.38 may have been a single high-variance rerun, not a reproducible result. Hidden rerun variance can swing ±0.3 LB on identical code.

Second, my fork used the `jeroencottaar/taaf-kaggle-source-share` bundle, while rokaiya's may have used a slightly different commit. The TAAF bundle has multiple forks with subtle differences.

The 0.79 floor is now the calibration point. V9's 1.10 is +0.31 over the floor, attributable to AGI_8+AGI_9 patches. V4's 1.06 is +0.27 over the floor, attributable to AGI_8+AGI_9 minus the synthesis prompt's -0.04 net effect.

## Next Version Plan

V11 was a private fork attempt that ERROR'd (competition wheelhouse not found). V12 was a Tufa milestone fork. V13 will use registration-only mode to skip the 9h commit and let the hidden rerun produce the real score.
