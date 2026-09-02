+++
title = "ARC3 V28 (LB 1.72): Text-Only Ablation - Hidden Games Prefer Image Modality"
date = 2026-08-27T05:45:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "ablation", "text-only"]
categories = ["ARC3 Dev Journal"]
summary = "V24 config with MULTIMODAL_CONTEXT changed from image to text. LB 1.72, -0.84 from V24's 2.56. Proves hidden games prefer image modality."
lb_score = "1.72"
version = "V28"
status = "ABLATION"
+++

## TL;DR

V24 config with MULTIMODAL_CONTEXT changed from image to text. LB 1.72, -0.84 from V24's 2.56. Proves hidden games prefer image modality.

## Context

V24 used image modality (MULTIMODAL_CONTEXT=current_grid with MULTIMODAL_UPSCALE=8). The hypothesis: text-only modality (MULTIMODAL_CONTEXT=text) would be faster (no vision token overhead) and might score similarly if the solver can read the board from text representation.

V28 was a controlled ablation: V24's exact config with one change, `MULTIMODAL_CONTEXT=text` instead of `current_grid`. If LB drops significantly, image modality matters. If LB stays near 2.56, text is sufficient.

## Technical Choice

Single-variable change: `MULTIMODAL_CONTEXT = 'text'` (was `'current_grid'`). All other patches (FP8 KV, MM_UPSCALE=8, WBC, RE cap, NG) preserved from V24.

The text representation is a 64x64 ASCII grid with color codes. The solver reads this as a string, not as an image. No vision tokens are generated.

## Parameter Decisions

| Parameter | V24 | V28 | Rationale |
|---|---|---|---|
| MULTIMODAL_CONTEXT | current_grid (image) | text | Ablation |
| MULTIMODAL_UPSCALE | 8 | 8 | unchanged (irrelevant for text) |
| FP8 KV cache | enabled | enabled | unchanged |
| model | Qwen3.8-27B-FP8 | Qwen3.8-27B-FP8 | unchanged |
| source bundle | anim bundle | anim bundle | unchanged |

## Local vs LB Score

- Local mean: not measured
- Baseline (previous version): V24 LB 2.56
- Local delta: n/a
- LB score: **1.72**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| FP8 KV cache | yes | ENABLE_FP8_KV=True |
| MULTIMODAL_UPSCALE=8 | yes | Patched MULTIMODAL_UPSCALE to 8 |
| Qwen3.8 model | yes | qwen3-8-27b-fp8-hf-snapshot |
| writable bundle copy | yes | refreshing writable bundle copy |
| reasoning effort cap | yes | reasoning_effort |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |

## Outcome Analysis

LB 1.72, -0.84 from V24's 2.56. Image modality matters significantly.

The 0.84 gap is the value of image modality over text modality, holding everything else constant. Image modality preserves spatial relationships that text representation loses. For ARC-AGI-3's cover predicates and co-location win conditions, spatial reasoning is essential.

V28 also recovered from V25's 1.42 to 1.72, confirming that the V24 patches (FP8 KV, WBC, RE cap, NG) are necessary and that the thtennant fork's loss of these patches caused V25's regression.

The lesson: V24's config (with image modality) is the local optimum. Text modality is a viable alternative for competitions where image tokens are too expensive, but for ARC-AGI-3, image wins.

## Next Version Plan

V30 was a synthesis AVO experiment. V31 will add NOOA modules on top of V24's config. Target: LB 3.0+ via cross-game learning.
