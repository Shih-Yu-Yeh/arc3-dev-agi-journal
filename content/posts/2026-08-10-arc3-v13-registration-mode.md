+++
title = "ARC3 V13 (LB 0.87): Tanaka Safety v1 + Registration Mode — 跳過 9 小時 Commit 的投機技巧"
date = 2026-08-10T12:48:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "TAAF", "registration-mode", "pipeline"]
categories = ["ARC3 Dev Journal"]
summary = "用 1 列 placeholder submission.parquet 跳過 9 小時 commit run。Hidden rerun 產出真實 LB 0.87。這是 pipeline 技巧，不是 solver 改善。"
lb_score = "0.87"
version = "V13"
status = "PIPELINE_TRICK"
+++


## 跳過 9 小時 commit

每次完整 commit run 在 RTX Pro 6000 上要 9 小時。每週 30 小時 GPU quota，限制每週只能 3 次完整提交。V13 用一個投機技巧繞過：commit run 階段寫 1 列 placeholder，跳過實際遊戲，讓 hidden rerun 產出真實分數。LB 0.87 跟 V10 純 baseline 在 hidden rerun variance 內等同，但 commit run 從 9 小時縮短到 2 分鐘。

## GPU quota 的瓶頸

每次完整 commit run 在 RTX Pro 6000 上要 9 小時。每週 30 小時 GPU quota，限制每週只能 3 次完整提交。瓶頸不是 solver，是跑公開 25 個遊戲的 commit run。

V13 測試一個 pipeline 技巧：commit run 階段寫 1 列 placeholder submission.parquet，跳過實際遊戲，讓 Kaggle 的 hidden rerun 產出真實分數。這個技巧在 `tanakaai24/arc3-qwen3-6-duck-lb117-safety-v1` 有文件說明。

## Registration mode 原理

Notebook 偵測 `KAGGLE_IS_COMPETITION_RERUN` env var。False（commit run）時寫 placeholder：

```python
submission = pd.DataFrame(
    data=[["1_0", "1", True, 1]],
    columns=["row_id", "game_id", "end_of_game", "score"],
)
submission.to_parquet("/kaggle/working/submission.parquet", index=False)
```

True（hidden rerun）時跑完整 TAAF solver。Commit run 2 分鐘完成（vs 9 小時），每次提交省 8h 58min GPU quota。

## commit run 從 9h 變 2 分鐘

| 參數 | V10 | V13 | 理由 |
|---|---|---|---|
| commit run 時長 | 9 小時 | 2 分鐘 | 跳過遊戲 |
| hidden rerun | 正常 | 正常 | 未變 |
| submission.parquet (commit) | 真實分數 | 1 列 placeholder | Registration only |
| solver | stock TAAF | stock TAAF (tanaka safety v1) | 同 V10 |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | 未變 |

## hidden rerun 產出真分

- Local mean: n/a（commit run 跳過）
- Baseline（上一版）: V10 LB 0.79
- Local delta: n/a
- LB score: **0.87**

## registration artifact 寫入

| Patch | 是否 fire? | Marker |
|---|---|---|
| registration mode | yes | Wrote Kaggle code-submission registration artifact |
| submission.parquet written | yes | submission.parquet |
| taaf_setup_env.json | yes | taaf_setup_env.json present |
| (solver patches) | n/a | Hidden rerun 跑真實 solver |

## 資源規劃的意涵

LB 0.87，比 V10 的 0.79 +0.08。在 hidden rerun variance 內，跟 V10 等同。

Registration-mode 技巧成功。Commit run 2 分鐘完成。Hidden rerun 跑真實 TAAF solver 並產出 LB 0.87。

這是 pipeline 技巧，不是 solver 改善。LB 0.87 是 tanaka safety v1 fork 的真實 hidden-game 分數，比 V10 純 baseline +0.08。

資源規劃的意涵：用這個技巧，我每天可以提交 3-5 個版本（受 hidden rerun queue 限制，不是 commit run 時長）。瓶頸從 GPU quota 轉到 Kaggle hidden rerun queue 深度。

這個技巧假設 hidden rerun 真的會跑 solver。如果 notebook 寫了 placeholder 但 hidden rerun 也寫 placeholder（例如 setup 失敗），LB 會回 0.00。V21 就是這個情況。

## V21 的前車之鑑

V14 會測 temperature 0.3（從 0.6 降）。判斷是：較低 temperature 產生更確定、可重現的動作，對嚴格 win condition 的 hidden games 有幫助。選 0.3 而不是 0.4 或 0.2，是因為 0.3 在保留多樣性與提升確定性之間取折衷。
