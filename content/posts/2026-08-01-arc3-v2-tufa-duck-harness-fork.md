+++
title = "ARC3 V2 (LB 0.87): Tufa Labs Duck Harness Fork - The Starting Point"
date = 2026-08-01T06:52:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-6"]
categories = ["ARC3 Dev Journal"]
summary = "Forked samrishb/sam-solver (Tufa Labs milestone 1.21). Stock Qwen3.6-27B-FP8. LB 0.87 within Tufa's expected 0.5-1.5 range."
lb_score = "0.87"
version = "V2"
status = "BASELINE"
+++

## TL;DR

Forked samrishb/sam-solver (Tufa Labs milestone 1.21). Stock Qwen3.6-27B-FP8. LB 0.87 within Tufa's expected 0.5-1.5 range.

## Context

After Day 0's pipeline failure, the priority was to establish a working baseline before adding any patches. The Tufa Labs duck harness (public kernel `samrishb/sam-solver`, LB 1.21 milestone) was the obvious choice: it had the full 5-step cycle (observe, reason, act, score, update), proper submission.parquet schema, and was already tested by the original team.

The question was not whether V2 would score well, but whether my fork would reproduce Tufa's result or diverge. Tufa Labs self-reported LB 0.5-1.5 variance on the same code, so any score in that range counts as a successful baseline.

## Technical Choice

Used the unmodified Tufa Labs source bundle (`jeroencottaar/taaf-kaggle-source-share`). Stock Qwen3.6-27B-FP8 model (`vrfai-qwen3-6-27b-fp8-hf-snapshot`). No patches, no grafts, no prompt addenda.

Three deliberate non-choices: did not upgrade to Qwen3.8 (would come at V17), did not enable FP8 KV cache (would come at V19), did not add MULTIMODAL_UPSCALE (would come at V24). The point was to verify that the baseline pipeline runs end-to-end and produces a real LB score.

## Parameter Decisions

| Parameter | Value | Rationale |
|---|---|---|
| model | Qwen3.6-27B-FP8 | Stock Tufa config |
| temperature | 0.6 | Stock |
| context window | 32768 | Stock |
| MULTIMODAL_UPSCALE | 4 (256x256) | Stock |
| n_passes | 1 | Stock |
| FP8 KV cache | disabled | Stock |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): n/a (first real run)
- Local delta: n/a
- LB score: **0.87**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## Outcome Analysis

LB 0.87 is within Tufa Labs' self-reported 0.5-1.5 range. The baseline pipeline works.

Three observations from the run:

First, the 9-hour wall clock is tight. Qwen3.6-27B-FP8 on a single RTX Pro 6000 with 32k context cannot complete all 110 hidden games within 9 hours. The give-up mechanism (max_runtime per game) kicked in on most games, capping contribution.

Second, the local temperature was 0.0 (not 0.6 as configured). The solver sets temperature=0 for action selection regardless of the configured `LOCAL_ANALYZER_TEMPERATURE`. The configured temperature only affects the reasoning step, not the action step. This distinction matters for later temperature experiments (V14).

Third, the vLLM server started successfully with `--max-model-len 65536` but the analyzer context window was only 32768. The model can handle 65k tokens, but the analyzer only feeds 32k. This gap is the target of V6 (context budget increase).

## Next Version Plan

Add a program synthesis system prompt patch on top of V2. Target: break above 1.0 LB by encouraging structured tool calls.
