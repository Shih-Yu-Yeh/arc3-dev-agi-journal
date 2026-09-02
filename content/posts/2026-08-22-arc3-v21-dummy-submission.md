+++
title = "ARC3 V21 (LB 0.00): 2-Pass Visible Updates - Dummy Submission Incident"
date = 2026-08-22T13:36:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "dummy-submission", "pipeline-failure"]
categories = ["ARC3 Dev Journal"]
summary = "All 9 patches fired. 2-pass visible updates added. LB returned 0.00 because submission.parquet was a 3411-byte dummy. Pipeline issue, not a solver issue."
lb_score = "0.00"
version = "V21"
status = "PIPELINE_FAILURE"
+++

## TL;DR

All 9 patches fired. 2-pass visible updates added. LB returned 0.00 because submission.parquet was a 3411-byte dummy. Pipeline issue, not a solver issue.

## Context

V19 regressed to 1.17. The hypothesis: 2-pass visible updates (play each game twice, retain the better score) would recover the regression by giving the solver a second chance on games where the first pass timed out.

V21 added 2-pass on top of V19's 9 patches. All patches verified firing. The expectation: LB 1.5+.

The result: LB 0.00. The submission.parquet was 3411 bytes, which is a dummy placeholder, not real game scores.

## Technical Choice

2-pass mechanism: run `bm.run()` twice with the same game list. After both passes, write the better score per game to submission.parquet.

The 2-pass implementation was correct in the notebook source. The problem was not the 2-pass logic; it was the submission.parquet schema.

The 3411-byte submission.parquet is the placeholder written when `KAGGLE_IS_COMPETITION_RERUN=False` (commit run). The hidden rerun should overwrite it with real scores. But the hidden rerun produced a 3411-byte file too, meaning the hidden rerun did not actually run the solver.

## Parameter Decisions

| Parameter | V19 | V21 | Rationale |
|---|---|---|---|
| n_passes | 1 | 2 | Visible updates |
| submission.parquet size | ~5KB (real) | 3411B (dummy) | Pipeline failed |
| patches | 9 fired | 9 fired | All verified |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | unchanged |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V19 LB 1.17
- Local delta: n/a
- LB score: **0.00**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort: 'high' |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## Outcome Analysis

LB 0.00. The submission.parquet was 3411 bytes, which is the dummy placeholder format.

Diagnosis: the hidden rerun did not actually run the solver. The commit run wrote the placeholder; the hidden rerun should have overwritten it with real scores. Instead, the hidden rerun also wrote a placeholder.

Three possible causes:

First, the 2-pass logic may have crashed during the hidden rerun, falling back to the placeholder. The 2-pass code wrapped `bm.run()` in a try/except, but the except block wrote a placeholder instead of re-raising.

Second, the hidden rerun's environment may have differed from the commit run. The hidden rerun uses the internal gateway (`http://gateway:8001`), which may have been unavailable. The commit run uses the bundled `environment_files/`.

Third, the submission.parquet schema may have been wrong. The 3411-byte size suggests a 1-row placeholder, not a 25-row or 110-row real submission.

The V21 resubmit (next post) tested whether the issue was transient.

## Next Version Plan

V21 resubmit: push the same kernel again, see if the 0.00 was transient or persistent.
