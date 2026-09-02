+++
title = "ARC3 V21 Resubmit (LB 0.00): Same Pit Twice - Dummy Submission Persists"
date = 2026-08-23T01:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "dummy-submission", "pipeline-failure"]
categories = ["ARC3 Dev Journal"]
summary = "Same V21 kernel resubmitted. LB still 0.00. Confirmed the dummy submission issue is not transient; it is a real bug in the 2-pass implementation."
lb_score = "0.00"
version = "V21 (resubmit)"
status = "PIPELINE_FAILURE"
+++

## TL;DR

Same V21 kernel resubmitted. LB still 0.00. Confirmed the dummy submission issue is not transient; it is a real bug in the 2-pass implementation.

## Context

V21 returned 0.00. The question: was this transient (Kaggle gateway hiccup) or persistent (real bug)?

V21 resubmit pushed the same kernel version again. If LB still returns 0.00, the issue is persistent and the 2-pass code has a real bug.

## Technical Choice

Identical kernel to V21. No code changes. The only difference is the submission timestamp.

## Parameter Decisions

| Parameter | V21 | V21 resubmit | Rationale |
|---|---|---|---|
| kernel version | v2 | v2 (same) | Identical |
| submission timestamp | 2026-08-22 13:36 | 2026-08-23 01:12 | Different |
| code changes | none | none | Identical |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V21 LB 0.00
- Local delta: 0.00
- LB score: **0.00**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| (same as V21) | yes | 9 patches fired |

## Outcome Analysis

LB 0.00 again. The dummy submission issue is persistent, not transient.

This confirms the 2-pass implementation has a real bug. The hidden rerun is not running the solver; it is writing a placeholder.

The fix: remove the 2-pass logic entirely. V23 will revert to 1-pass with the anim bundle.

## Next Version Plan

Abandon 2-pass. V23 will use the anim bundle (`jakobbrggen/taaf-kaggle-source-anim-20260807-anim`) with 1-pass and the Qwen3.8 model patch.
