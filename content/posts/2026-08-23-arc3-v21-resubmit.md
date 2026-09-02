+++
title = "ARC3 V21 RESUBMIT (LB 0.00): 同一坑踩兩次 — Dummy Submission 追蹤"
date = 2026-08-23T01:12:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "dummy-submission", "pipeline-failure"]
categories = ["ARC3 Dev Journal"]
summary = "同一份 V21 kernel 重 submit。LB 還是 0.00。確認 dummy submission 問題不是 transient；是 2-pass 實作的真 bug。"
lb_score = "0.00"
version = "V21 (resubmit)"
status = "PIPELINE_FAILURE"
+++

## TL;DR

同一份 V21 kernel 重 submit。LB 還是 0.00。確認 dummy submission 問題不是 transient；是 2-pass 實作的真 bug。

## Context

V21 回 0.00。問題：是 transient (Kaggle gateway hiccup) 還是 persistent (真 bug)？

V21 resubmit 再 push 同一個 kernel version。如果 LB 還是 0.00，問題是 persistent，2-pass code 有真 bug。

## 技術選擇

跟 V21 完全相同的 kernel。沒改 code。唯一差異是 submission 時間戳。

## 參數決策

| 參數 | V21 | V21 resubmit | 理由 |
|---|---|---|---|
| kernel version | v2 | v2 (同) | 完全相同 |
| submission 時間戳 | 2026-08-22 13:36 | 2026-08-23 01:12 | 不同 |
| code 改動 | 無 | 無 | 完全相同 |

## Local vs LB Score

- Local mean: 未量測
- Baseline (上一版): V21 LB 0.00
- Local delta: 0.00
- LB score: **0.00**

## Patch 驗證

| Patch | 是否 fire? | Marker |
|---|---|---|
| (同 V21) | yes | 9 個 patch fire |

## 結果分析

LB 還是 0.00。Dummy submission 問題是 persistent，不是 transient。

確認 2-pass 實作有真 bug。Hidden rerun 沒跑 solver；它在寫 placeholder。

修法：完全移除 2-pass 邏輯。V23 會退回 1-pass + anim bundle。

## 下一版計畫

放棄 2-pass。V23 會用 anim bundle (`jakobbrggen/taaf-kaggle-source-anim-20260807-anim`) + 1-pass + Qwen3.8 model patch。
