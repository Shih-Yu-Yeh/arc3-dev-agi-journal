+++
title = "ARC3 V2 (LB 0.87): Tufa Labs Duck Harness Fork — 我的 ARC3 起點"
date = 2026-08-01T06:52:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-6"]
categories = ["ARC3 Dev Journal"]
summary = "Fork 自 samrishb/sam-solver (Tufa Labs milestone 1.21)。Stock Qwen3.6-27B-FP8。LB 0.87，落在 Tufa 預期的 0.5-1.5 範圍內。"
lb_score = "0.87"
version = "V2"
status = "BASELINE"
+++

## TL;DR

Fork 自 samrishb/sam-solver (Tufa Labs milestone 1.21)。Stock Qwen3.6-27B-FP8。LB 0.87，落在 Tufa 預期的 0.5-1.5 範圍內。

## Context

Day 0 pipeline 失敗後，第一要務是在加任何 patch 前先建立可運作的 baseline。Tufa Labs duck harness (公開 kernel `samrishb/sam-solver`，LB 1.21 milestone) 是顯然選擇：它有完整的 5 步循環 (observe、reason、act、score、update)、正確的 submission.parquet schema，且原作者已測試過。

問題不是 V2 會不會得分，而是我的 fork 能否重現 Tufa 的結果還是會發散。Tufa Labs 自報同一份 code LB 變動 0.5-1.5，所以任何落在此範圍的分數都算 baseline 成功。

## 技術選擇

使用未修改的 Tufa Labs source bundle (`jeroencottaar/taaf-kaggle-source-share`)。Stock Qwen3.6-27B-FP8 model (`vrfai-qwen3-6-27b-fp8-hf-snapshot`)。無 patch、無 graft、無 prompt addendum。

三個刻意的「不做」：不升級到 Qwen3.8 (V17 才做)、不啟用 FP8 KV cache (V19 才做)、不加 MULTIMODAL_UPSCALE (V24 才做)。

重點是驗證 baseline pipeline 端到端跑通並產出真實 LB 分數。

## 參數決策

| 參數 | 值 | 理由 |
|---|---|---|
| model | Qwen3.6-27B-FP8 | Stock Tufa 配置 |
| temperature | 0.6 | Stock |
| context window | 32768 | Stock |
| MULTIMODAL_UPSCALE | 4 (256x256) | Stock |
| n_passes | 1 | Stock |
| FP8 KV cache | disabled | Stock |

## Local vs LB Score

- Local mean: 未量測
- Baseline (上一版): n/a (第一次真實 run)
- Local delta: n/a
- LB score: **0.87**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## 結果分析

LB 0.87 落在 Tufa Labs 自報的 0.5-1.5 範圍內。Baseline pipeline 運作正常。

三個觀察：

第一，9 小時 wall clock 很緊。Qwen3.6-27B-FP8 在單張 RTX Pro 6000 + 32k context 下無法在 9 小時內跑完所有 110 個 hidden games。Give-up 機制 (每遊戲 max_runtime) 在大多數遊戲上啟動，限制貢獻分數。

第二，local temperature 是 0.0 (不是設定的 0.6)。Solver 在 action selection 階段固定用 temperature=0，與設定的 `LOCAL_ANALYZER_TEMPERATURE` 無關。設定的 temperature 只影響 reasoning 階段，不影響 action 階段。這個區別對後續的 temperature 實驗 (V14) 很重要。

第三，vLLM server 以 `--max-model-len 65536` 啟動成功，但 analyzer context window 只有 32768。Model 能處理 65k tokens，但 analyzer 只餵 32k。這個缺口是 V6 (context budget 增加) 的目標。

## 下一版計畫

在 V2 上加 program synthesis system prompt。目標：靠鼓勵結構化 tool call 突破 1.0 LB。
