+++
title = "ARC3 V6 (LB 0.75): Context Budget 32768 → 49152 — 過頭的代價 -0.31"
date = 2026-08-04T02:19:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "context-window", "regression"]
categories = ["ARC3 Dev Journal"]
summary = "把 analyzer context window 從 32k 加到 49k tokens (+50%)。LB 從 1.06 掉到 0.75 (-0.31)。Local mean 0.58，預測會改善。第一個硬教訓：local mean ≠ LB score。"
lb_score = "0.75"
version = "V6"
status = "REGRESSION"
+++

## TL;DR

把 analyzer context window 從 32k 加到 49k tokens (+50%)。LB 從 1.06 掉到 0.75 (-0.31)。Local mean 0.58，預測會改善。第一個硬教訓：local mean ≠ LB score。

## Context

V4 的 context window 是 32768 tokens。vLLM server 跑 `--max-model-len 65536`。中間有 32k 的缺口。假設：更大的 context window 讓 solver 保留更多遊戲歷史，對多 level 遊戲有幫助。

Local dry-run 在 6 個公開遊戲上支持這個假設：local mean 從 V4 的 0.45 升到 0.58 (+29%)。我 push 到 LB 期待類似改善。結果是相反的。

## 技術選擇

單一變數改動：`ANALYZER_CONTEXT_WINDOW = 49152` (原 32768)。+50% 增加。其他不動。

選 49152 而不是 65536 是刻意的：留 headroom 給 action prompt 和 observation tokens。Analyzer context window 在遊戲歷史、system prompt、當前 observation 之間共享。填到 65k 會沒有空間放 response。

## 參數決策

| 參數 | V4 | V6 | 理由 |
|---|---|---|---|
| ANALYZER_CONTEXT_WINDOW | 32768 | 49152 | +50% 保留更多遊戲歷史 |
| vLLM --max-model-len | 65536 | 65536 | 未變 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | 未變 |
| temperature | 0.6 | 0.6 | 未變 |
| MULTIMODAL_UPSCALE | 4 | 4 | 未變 |

## Local vs LB Score

- Local mean: 0.58
- Baseline (上一版): V4 local 0.45
- Local delta: +0.13 (+29%)
- LB score: **0.75**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 49152 |
| vLLM server started | yes | vLLM server ready |
| submission.parquet written | yes | submission.parquet |
| give-up mechanism | yes | max_runtime |

## 結果分析

LB 0.75，比 V4 的 1.06 -0.31。Local mean 改善 +29%，LB 退步 -29%。第一個硬證據：local mean 與 hidden games 的 LB score 無相關。

三個診斷：

第一，local 公開遊戲短 (平均 6-8 levels，每 level 30-50 actions)。49k context window 輕鬆容納整個遊戲歷史。Solver 保留所有資訊，對熟悉的遊戲有幫助。

第二，hidden games 較長。25 個公開遊戲的 `baseline_actions` 總和是 17135 (平均每遊戲 685)，但 hidden games 可能有更長的 action 序列。49k context 下 solver 可能保留了早期 levels 已無關聯的雜訊，稀釋了對當前 level 的注意力。

第三，更大 context 增加每次 inference call 的延遲。9 小時 wall clock + 110 個遊戲，每多一秒會累積。Solver 可能在更多遊戲上 timeout，降低貢獻。

Local mean 改善是真的，但量測在錯誤的分佈上。公開遊戲偏好更多 context；hidden games 偏好更少。這是 §9 教訓：local mean 在公開遊戲上的改善不會轉移到 hidden games。

## 下一版計畫

把 context window 退回 32768 (V7 是私下測試)。V9 會重製 yw8837 的 LB-1.17 config 作為 controlled comparison。
