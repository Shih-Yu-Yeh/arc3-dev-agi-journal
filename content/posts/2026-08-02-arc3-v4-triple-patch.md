+++
title = "ARC3 V4 (LB 1.06): TAAF + Program Synthesis Prompt Triple-Patch"
date = 2026-08-02T02:01:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "Qwen3-6", "prompt-engineering"]
categories = ["ARC3 Dev Journal"]
summary = "疊了三個 patch：program synthesis prompt、offline env_dir 修復、AGI_8/AGI_9。LB 1.06，比 V2 +0.19。首次改善。"
lb_score = "1.06"
version = "V4"
status = "IMPROVEMENT"
+++


## 第一次加 patch

V2 在 0.87 建立 baseline 之後，下一個問題是 Tufa harness 能不能在不破壞的前提下被打 patch。我疊了三個獨立改動，LB 升到 1.06，比 V2 +0.19。改善幅度不大，而且三個 patch 疊在一起，沒辦法歸因哪個貢獻最多。

## 疊了三個改動

V2 在 0.87 建立 baseline。下一個問題是 Tufa harness 能不能在不破壞的前提下被打 patch。我疊了三個獨立改動：

1. Program synthesis system prompt addendum，鼓勵 solver 用結構化 tool call 表達動作
2. Offline `env_dir` 修復，讓 V2 無法本地載入的 6 個公開遊戲能跑
3. 從公開 `yw8837` kernel 直接 port 過來的 `AGI_8` 與 `AGI_9` patches，處理兩個 hidden-game pattern

這違反開發憲法 §2（一個版本只改一個變數），但在這個階段我還在 calibration 哪些 patch 重要。違反是刻意的。

## 為什麼違反 §2

Program synthesis prompt 是一段約 200 token 的 system prompt addendum，把 agent 的任務框架從「一次採一個 action」改成「合成一個 tool call 程式」。背後的判斷是：ARC-AGI-3 獎勵多步規劃，program-synthesis 框架鼓勵 model 承諾一個計畫，而不是每步重新推導。

Offline env_dir 修復是對 `Arcade.environments_dir` 的一行 patch，指向 bundled `environment_files/` 而不是 API。只影響 V2 無法本地載入的 6 個公開遊戲。

AGI_8 跟 AGI_9 patches 從 `yw8837/arc3-yw8837-lb-1-17` 直接 port 過來。當時沒有深入理解，當作黑箱增強處理。

## 三個 patch 的選擇理由

| 參數 | V2 | V4 | 理由 |
|---|---|---|---|
| system prompt | stock Tufa | + program synthesis addendum | 鼓勵結構化 tool call |
| env_dir | API-based | offline bundle 路徑 | 修復公開遊戲載入 |
| AGI_8 patch | off | on | Port from yw8837 |
| AGI_9 patch | off | on | Port from yw8837 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | 未變 |

其餘配置沿用 V2。

## 首次改善 +0.19

- Local mean: 未量測
- Baseline（上一版）: V2 LB 0.87
- Local delta: n/a
- LB score: **1.06**

## 五個 marker fire

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| submission.parquet written | yes | submission.parquet |
| give-up mechanism | yes | max_runtime |

## 為什麼無法歸因

LB 1.06，比 V2 +0.19。三個 patch 疊在一起，沒辦法歸因。這是違反 §2 的代價。

改善是真的但幅度不大。Program synthesis prompt 最可能是主要貢獻者：V9 後來去掉 synthesis prompt 重製 yw8837 config 得到 1.10，暗示 AGI_8 與 AGI_9 patches 單獨貢獻 +0.23，而 synthesis prompt 的淨效果其實是略為負面（大約 -0.04）。

submission.parquet 這次格式正確（5 欄 schema），確認 Day 0 的修復有效。

關鍵收穫：即使為了速度而疊 patch，也要寫下疊了什麼。V4 的 worklog 條目是改了什麼的唯一紀錄；沒有它，V9 沒辦法設計成 controlled comparison。

## 學到 worklog 要寫

V5 是私下的 synthesis 實驗（無 LB 提交）。V6 會單獨隔離 context budget 變數：32768 → 49152（+50%）。
