+++
title = "ARC3 序言 — Kaggle 平台與 ARC Prize 2026 簡介"
date = 2026-07-30T00:00:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "prologue"]
categories = ["ARC3 Dev Journal"]
summary = "進入開發紀錄前的背景說明：Kaggle 平台的角色、ARC Prize 2026 想推動的問題、以及我進場前對規則的理解。"
lb_score = "n/a"
version = "Prologue"
status = "INTRO"
+++

## 摘要

這篇是開發紀錄的起點。正文從 V2 開始，每篇對應一次 leaderboard 提交，記錄技術選擇、參數決策、分數變化、以及背後的判斷。在進入正文之前，先把兩件事講清楚：Kaggle 這個平台在做什麼，以及 ARC Prize 2026 (ARC-AGI-3) 想測的東西。

## 1. Kaggle 平台

Kaggle 是 Google 旗下的數據科學競賽平台，全球超過一千萬註冊用戶。競賽分三類：Featured（高額獎金）、Research（學術導向）、Playground（練習用）。

提交方式也分兩種。Research competition 可以直接上傳 CSV 預測結果；Code competition 則要求提交 notebook，由 Kaggle server 在 hidden test set 上重新執行計分。ARC-AGI-3 屬於後者，而且隱藏了 110 個遊戲，只透過內部 gateway 提供，參賽者無法事先看到遊戲源碼。這個設計確保選手不能用 hardcode 答案作弊。

## 2. ARC Prize 2026 — ARC-AGI-3

ARC Prize 由 François Chollet（Keras 作者）在 2019 年發起，目的是推動 AGI 評測標準。前三代 ARC 都是「圖形推理」題——模型看一張圖，輸出對應的轉換圖。ARC-AGI-3 是 2026 年的第三代，**首次把題目改成互動式**：模型不再只看圖答題，而是要像人類玩家一樣實際操作遊戲。

每場競賽給 9 小時實際運行時間（wall clock），在 110 個 hidden games 上各玩一輪。計分方式是 RHAE（Relative Human Action Efficiency），比較 agent 與人類 baseline 的動作效率——動作越少越好。

官方想驗證的核心問題是：LLM 能否在完全陌生的環境中，透過觀察、假設、驗證、修正的循環，達到人類水準的問題解決能力？這個能力跟「背誦訓練資料中的知識」是兩件事，比較接近「學習如何學習」。獎金池 850,000 美金，目前有 2708 隊伍參賽。

## 3. 進場前的理解與規則

**Kaggle Code Competition 的兩階段流程**

提交 notebook 之後，Kaggle 會跑兩階段：

| 階段 | 內容 | 時間 | 計分 |
|------|------|------|------|
| Commit run | 在公開 25 個遊戲上跑通 notebook，產出 submission.parquet | 計入 9 小時 wall clock | 不計分 |
| Hidden rerun | 用內部 gateway 提供 110 個 hidden games，重新執行計分 | 計入 9 小時 wall clock | 計入 LB |

兩階段都消耗 9 小時 wall clock 預算，而且共用每週 30 小時的 GPU quota。

**ARC-AGI-3 的規則要點**

- 110 個 hidden games：遊戲源碼不公開，只透過 HTTP API 提供 frame data 與 available_actions
- 1 pass：每局只能玩一次，不能 best-of-N 重試
- 9 小時 wall clock：含 vLLM 啟動、model load、推理時間
- RHAE scoring：相對於人類 baseline 的動作效率比
- 每場遊戲包含多個 levels：25 個公開遊戲總共 183 個 levels
- 動作類型分三類：click、keyboard、keyboard_click

**計分公式**

```
score = sum over levels of (baseline_actions / actual_actions, capped per level)
```

完成第 0 個 level 的分數上限大約是 3.52。要突破天花板，必須完成更多 levels。

**我進場時的目標**

驗證 Qwen3.8-27B-FP8 加上視覺細節提升，能否在單張 RTX Pro 6000 上達到 LB > 2.0。後續 21 篇紀錄會逐步展開這個目標如何被驗證、推翻、修正，最後在 V24 達到 2.56 的個人最佳。
