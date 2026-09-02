+++
title = "ARC3 V17 (LB 1.43): Qwen3.8-27B-FP8 + Visual Updates - First Breakthrough Above 1.10"
date = 2026-08-17T07:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "FP8", "visual-updates"]
categories = ["ARC3 Dev Journal"]
summary = "Upgraded from Qwen3.6-27B-FP8 to Qwen3.8-27B-FP8. Added visual update mechanism. LB 1.43, +0.33 over V15. First breakthrough above the 1.10 ceiling."
lb_score = "1.43"
version = "V17"
status = "BREAKTHROUGH"
+++

## TL;DR

Upgraded from Qwen3.6-27B-FP8 to Qwen3.8-27B-FP8. Added visual update mechanism. LB 1.43, +0.33 over V15. First breakthrough above the 1.10 ceiling.

## Context

V9-V15 had been stuck in the 0.79-1.10 range. The Qwen3.6 model had reached its ceiling. Two changes were needed: a better model and a better way to feed visual state.

Qwen3.8-27B-FP8 was released in early August 2026 with improved tool-call accuracy. The `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle had also just been published, with animation-awareness and visual update support.

V17 stacked both changes. This violates §2 (one variable per version), but the model upgrade was the dominant hypothesis.

## Technical Choice

Model upgrade: Qwen3.6-27B-FP8 to Qwen3.8-27B-FP8 (`jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`). Qwen3.8 has better tool-call parsing (`qwen3_coder` parser) and improved reasoning_effort control.

Visual updates: switched to `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle. This bundle includes animation-awareness, which detects when the game board is mid-animation and waits for the final state before reading.

Three patches verified firing:
- Qwen3.8 model loaded (confirmed in vLLM startup log: `--model /kaggle/input/datasets/jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`)
- Noop guard verified (`hard_noop_guard = True`)
- Visual updates mechanism loaded (anim bundle imported)

## Parameter Decisions

| Parameter | V15 | V17 | Rationale |
|---|---|---|---|
| model | Qwen3.6-27B-FP8 | Qwen3.8-27B-FP8 | Better tool-call accuracy |
| source bundle | dvm fork | jakobbrggen anim bundle | Animation-awareness |
| tool call parser | qwen3 | qwen3_coder | Qwen3.8 native |
| reasoning parser | qwen3 | qwen3 | unchanged |
| temperature | 0.6 | 0.6 | unchanged |
| MULTIMODAL_UPSCALE | 4 | 4 | unchanged (V24 will upgrade) |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V15 LB 0.80
- Local delta: n/a
- LB score: **1.43**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| Qwen3.8 model loaded | yes | Qwen/Qwen3.8-27B-FP8 |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## Outcome Analysis

LB 1.43, +0.63 over V15's 0.80. First breakthrough above the 1.10 ceiling.

Attribution is impossible because both changes (model + bundle) stacked. But circumstantial evidence:

- Qwen3.8 has better tool-call accuracy per the release notes. This matters for ARC-AGI-3 because the solver issues many structured tool calls per action.
- The anim bundle's animation-awareness prevents reading mid-animation frames. Without this, the solver may see a partial state and issue wrong actions.
- `hard_noop_guard = True` was set by the anim bundle, not by my patches. This is the first version where noop guard was verified.

The 0.63 improvement likely comes from all three: model upgrade + animation-awareness + noop guard. The dominant contributor is probably the model upgrade, but this cannot be confirmed without a controlled ablation.

## Next Version Plan

V18 was a private jakob q38 pure test. V19 will add the P2+P3 patches (FP8 KV cache, noop guard wrapper, reasoning effort cap) on top of V17's Qwen3.8 config.
