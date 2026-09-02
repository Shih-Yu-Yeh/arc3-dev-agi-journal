+++
title = "ARC3 序言 - Kaggle 平台與 ARC Prize 2026 簡介"
date = 2026-07-30T00:00:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "prologue"]
categories = ["ARC3 Dev Journal"]
summary = "Kaggle 平台作用、ARC Prize 2026 - ARC-AGI-3 官方意圖、初步理解與規則。"
lb_score = "n/a"
version = "Prologue"
status = "INTRO"
+++

## TL;DR

進入 ARC3 開發紀錄正文前的背景說明：Kaggle 是什麼、ARC Prize 2026 想解決什麼問題、我進場前對規則的理解。

## 1. Kaggle 平台簡介

Kaggle 是 Google 旗下的數據科學競賽平台，全球超過一千萬註冊用戶。它提供三類競賽：Featured（高額獎金）、Research（學術導向）、Playground（練習用）。提交方式分兩種：Research competition 直接上傳 CSV 預測結果；Code competition 則要求提交 notebook，由 Kaggle server 在 hidden test set 上 rerun 計分。ARC-AGI-3 屬於後者，且隱藏 110 個遊戲透過內部 gateway 提供，參賽者無法事先看到遊戲源碼。

## 2. ARC Prize 2026 - ARC-AGI-3 簡介

ARC Prize 由 François Chollet（Keras 作者）於 2019 年發起，目的是推動 AGI 評測標準。ARC-AGI-3 是 2026 年第三代，**首次從「圖形推理」轉向「互動式 AI agent」**：模型不再只看一張圖答題，而是要像人類玩家一樣實際操作遊戲。每場競賽給予 9 小時 wall clock，在 110 個 hidden games 上各玩一輪，採 RHAE（Relative Human Action Efficiency）計分，比較 agent 與人類 baseline 的動作效率。

官方想驗證的核心命題：LLM 能否在完全陌生的環境中，透過觀察、假設、驗證、修正的循環，達成人類水準的問題解決能力？這是邁向 AGI 的關鍵一步——不是背誦訓練資料中的知識，而是學習如何學習。獎金池 850,000 美金，目前 2708 隊伍參賽。

## 3. 初步理解與規則

進入比賽前我對 Kaggle 與 ARC-AGI-3 的理解：

**Kaggle Code Competition 運作方式**

提交 notebook 後流程分兩階段：

| 階段 | 動作 | 時間 | 計分 |
|------|------|------|------|
| Commit run | 在公開 25 個遊戲上跑通 notebook，產出 submission.parquet | 計入 9h wall clock | 不計分 |
| Hidden rerun | 用內部 gateway:8001 提供 110 個 hidden games，rerun 計分 | 計入 9h wall clock | 計入 LB |

兩次 run 都消耗 9 小時 wall clock 預算，且 GPU quota 共用。

**ARC-AGI-3 規則**

- 110 hidden games（遊戲源碼不公開，僅透過 HTTP API 提供 frame data + available_actions）
- 1 pass（每局只能玩一次，不能 best-of-N 重試）
- 9 小時 wall clock（含 vLLM 啟動、model load、推理時間）
- RHAE scoring（相對於人類 baseline 的動作效率比）
- 每場遊戲包含多個 levels（25 個公開遊戲共 183 levels）
- 動作 tags：click / keyboard / keyboard_click

**評分公式**

score = sum over levels of (baseline_actions / actual_actions, capped per level)

完成第 0 level 的分數上限約 3.52；要突破天花板必須完成更多 levels。

**我的目標**

驗證 Qwen3.8-27B-FP8 + 視覺細節提升能否在單張 RTX Pro 6000 上達到 LB > 2.0。後續 21 篇紀錄會逐步展開這個目標如何被驗證、推翻、修正，最終在 V24 達到 2.56 的個人最佳。
