+++
title = "ARC3 V17 (LB 1.43): Qwen3.8-27B-FP8 + Visual Updates — 突破 1.10 天花板"
date = 2026-08-17T07:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-8", "FP8", "visual-updates"]
categories = ["ARC3 Dev Journal"]
summary = "從 Qwen3.6 升級到 Qwen3.8-27B-FP8。加 anim bundle 的 animation-awareness 與 noop guard。LB 1.43，比 V15 +0.63。首次突破 1.10 天花板。"
lb_score = "1.43"
version = "V17"
status = "BREAKTHROUGH"
+++

## 摘要

V9 到 V15 一直卡在 0.79-1.10 區間。Qwen3.6 model 已經到天花板，需要兩個改動：更好的 model 跟更好的視覺狀態處理。V17 把 model 從 Qwen3.6-27B-FP8 升級到 Qwen3.8-27B-FP8，同時切到 `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle（含 animation-awareness 與 hard noop guard）。LB 1.43，比 V15 +0.63，首次突破 1.10 天花板。

## Context

V9 到 V15 一直卡在 0.79-1.10 區間。Qwen3.6 model 已到天花板。需要兩個改動：更好的 model 與更好的視覺狀態餵法。

Qwen3.8-27B-FP8 在 2026 年 8 月初發布，release notes 強調 tool-call 準確度提升。`jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle 也剛發布，含 animation-awareness 與 visual update 支援。

V17 疊了兩個改動。這違反 §2（一個版本只改一個變數），但 model 升級是主要假設。

## 技術選擇

Model 升級：Qwen3.6-27B-FP8 → Qwen3.8-27B-FP8（`jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`）。Qwen3.8 有更好的 tool-call parsing（`qwen3_coder` parser）與 reasoning_effort 控制。

Visual updates：切到 `jakobbrggen/taaf-kaggle-source-anim-20260807-anim` bundle。這個 bundle 含 animation-awareness，能偵測遊戲盤面是否在動畫中，等到最終狀態才讀。

三個已驗證 fire 的 patch：

- Qwen3.8 model 載入（vLLM 啟動 log 確認：`--model /kaggle/input/datasets/jakobbrggen/qwen3-8-27b-fp8-hf-snapshot`）
- Noop guard 驗證（`hard_noop_guard = True`）
- Visual updates 機制載入（anim bundle imported）

## 參數決策

| 參數 | V15 | V17 | 理由 |
|---|---|---|---|
| model | Qwen3.6-27B-FP8 | Qwen3.8-27B-FP8 | 更好的 tool-call 準確度 |
| source bundle | dvm fork | jakobbrggen anim bundle | Animation-awareness |
| tool call parser | qwen3 | qwen3_coder | Qwen3.8 原生 |
| reasoning parser | qwen3 | qwen3 | 未變 |
| temperature | 0.6 | 0.6 | 未變 |
| MULTIMODAL_UPSCALE | 4 | 4 | 未變（V24 才升級） |

## Local vs LB Score

- Local mean: 未量測
- Baseline（上一版）: V15 LB 0.80
- Local delta: n/a
- LB score: **1.43**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| Qwen3.8 model loaded | yes | Qwen/Qwen3.8-27B-FP8 |
| noop guard verified | yes | hard_noop_guard = True |
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## 結果分析

LB 1.43，比 V15 的 0.80 +0.63。首次突破 1.10 天花板。

歸因上沒辦法很乾淨，因為兩個改動（model + bundle）疊在一起。但旁證上有幾個：

Qwen3.8 有更好的 tool-call 準確度（per release notes）。這對 ARC-AGI-3 重要，因為 solver 每個動作要發很多結構化 tool call。

Anim bundle 的 animation-awareness 防止讀到動畫中的盤面。沒有這個，solver 可能看到部分狀態而發錯動作。

`hard_noop_guard = True` 是 anim bundle 設的，不是我的 patch。這是第一次 noop guard 被驗證。

0.63 改善可能來自三者：model 升級、animation-awareness、noop guard。主要貢獻者可能是 model 升級，但沒有 controlled ablation 沒辦法確認。

## 下一版計畫

V18 是私下 jakob q38 pure 測試。V19 會在 V17 的 Qwen3.8 config 上加 P2+P3 patches（FP8 KV cache、noop guard wrapper、reasoning effort cap）。
