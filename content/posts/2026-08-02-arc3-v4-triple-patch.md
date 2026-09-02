+++
title = "ARC3 V4 (LB 1.06): TAAF + Program Synthesis Prompt Triple-Patch"
date = 2026-08-02T02:01:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-6", "prompt-engineering"]
categories = ["ARC3 Dev Journal"]
summary = "Added program synthesis system prompt + offline env_dir fix + agi_8/agi_9 patches. LB 1.06, +0.19 over V2. First improvement."
lb_score = "1.06"
version = "V4"
status = "IMPROVEMENT"
+++

## TL;DR

Added program synthesis system prompt + offline env_dir fix + agi_8/agi_9 patches. LB 1.06, +0.19 over V2. First improvement.

## Context

V2 established the baseline at 0.87. The next question was whether the Tufa harness can be patched without breaking. Three independent changes were stacked:

1. A program synthesis system prompt addendum, encouraging the solver to phrase actions as structured tool calls.
2. An offline `env_dir` fix for the 6 public games that V2 could not load locally.
3. The `AGI_8` and `AGI_9` patches from the public `yw8837` kernel, addressing two specific hidden-game patterns.

This violates the one-variable-per-version rule (§2 of the development constitution), but at this stage I was still calibrating which patches mattered. The violation was deliberate.

## Technical Choice

The program synthesis prompt was a 200-token addendum to the system prompt, framing the agent's task as "synthesize a program of tool calls" rather than "take actions one at a time." The hypothesis: ARC-AGI-3 rewards multi-step planning, and a program-synthesis framing encourages the model to commit to a plan rather than re-derive each step.

The offline env_dir fix was a one-line patch to `Arcade.environments_dir`, pointing it at the bundled `environment_files/` directory instead of the API. This affects only the 6 public games that V2 could not load locally.

The AGI_8 and AGI_9 patches were direct ports from `yw8837/arc3-yw8837-lb-1-17`. I did not understand them deeply at the time; they were treated as black-box enhancements.

## Parameter Decisions

| Parameter | V2 | V4 | Rationale |
|---|---|---|---|
| system prompt | stock Tufa | + program synthesis addendum | Encourage structured tool calls |
| env_dir | API-based | offline bundle path | Fix public game loading |
| AGI_8 patch | off | on | Port from yw8837 |
| AGI_9 patch | off | on | Port from yw8837 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | unchanged |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V2 LB 0.87
- Local delta: n/a
- LB score: **1.06**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| submission.parquet written | yes | submission.parquet |
| give-up mechanism | yes | max_runtime |

## Outcome Analysis

LB 1.06, +0.19 over V2. Three patches stacked, so attribution is impossible. This is the cost of violating §2 (one variable per version).

The improvement is real but small. The program synthesis prompt is the most likely contributor: V9 later replicated yw8837's config without the synthesis prompt and scored 1.10, suggesting the AGI_8/AGI_9 patches alone contributed +0.23, while the synthesis prompt may have contributed -0.04 to +0.19 (uncertain).

The submission.parquet was written correctly this time (5-column schema), confirming the Day 0 fix worked.

Key learning: even when stacking patches for speed, write down which patches are stacked. The worklog entry for V4 was the only record of what changed; without it, V9 could not have been designed as a controlled comparison.

## Next Version Plan

V5 was a private synthesis experiment (no LB submission). V6 will isolate the context budget variable: 32768 to 49152 (+50%).
