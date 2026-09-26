+++
title = "ARC3 Dev Journal"
date = 2026-07-31T00:00:00+08:00
draft = false
+++

# ARC Prize 2026 (ARC-AGI-3) — 開發紀錄

33 天。21 次公開 leaderboard 提交。33 個私下實驗版本。一個問題：單張 RTX Pro 6000 + Qwen3.8-27B-FP8 能否在 110 個 hidden 互動推理遊戲上達到 LB 2.0+？

答案出乎意料：可以，但方向不是我預期的。

## 關鍵結果

**V24 在 2026-08-25 達到 LB 2.56**，方法是將 `MULTIMODAL_UPSCALE` 從 4 升到 8（256x256 改 512x512 vision rendering）。其他所有改動——modules、grafts、prompt addenda、context budget、temperature tuning——都退步或沒有可量測效果。

![V24 架構圖](/arc3-dev-agi-journal/images/v24-arch.png)

## 這份 Journal 是什麼

這份 journal 每篇記錄一次 leaderboard 提交。結構一致：

- **Context** — 為什麼這個方向，什麼假設驅動它
- **技術選擇** — 改了什麼，為什麼選這個方案而非其他
- **參數決策** — 參數表，上一版 vs 當前版，含理由
- **Local vs LB Score** — local mean 與 leaderboard 的差距，含診斷
- **Patch 驗證** — 4 層檢查（syntax、marker print、vLLM config、events.jsonl）
- **結果分析** — 發生什麼、為什麼、下一步

這不是教學文件，是決策、證據、修正的 logbook。

## 為什麼公開

我犯了三個昂貴的錯誤，每個都花了完整的 30 小時 cycle：

1. **V31**：聲稱 7 個 modules 已安裝，實際 0 個 fire。一個 bare `except Exception: pass` 把 `AttributeError` 藏了整個 9 小時 run。
2. **V35.1**：step_env 有 2443 個 `UnboundLocalError`。Kernel 仍完成並提交，但每個 step 都靜默崩潰。
3. **V25**：在 V24 的 winning config 上加 7 個 grafts，過程中掉了 4 個關鍵 patch（FP8 KV、writable bundle copy、reasoning effort cap、noop guard）。LB 從 2.56 掉到 1.42。

共同 pattern：退步藏在「code 聲稱的」與「runtime 實際做的」之間的 gap。這份 journal 試圖讓那個 gap 可見——為了我自己在未來競賽，也為其他在 build AI agent submission for Kaggle code competition 的人。

## 從這裡開始

- [序言 — Kaggle 平台與 ARC-AGI-3 簡介](/posts/2026-07-30-arc3-prologue-kaggle-and-arc-agi-3/)
- [LB Progress 圖表與表格](/lb-progress/) — 21 次提交一覽
- [Lessons Learned](/lessons-learned/) — 16 條開發憲法 + 5 個最大技術發現
- [Abbreviations](/abbreviations/) — 縮寫對照表
- [About](/about/) — 作者簡介、技術棧、聯絡
- [Posts](/posts/) — 每篇對應一次提交，按時間順序

## 依結果快速導覽

**最佳**：[V24 — LB 2.56 (winner)](/posts/2026-08-25-arc3-v24-mm-upscale8-winner/)

**最差**：[V21 — LB 0.00 (dummy submission)](/posts/2026-08-22-arc3-v21-dummy-submission/)

**靜默失敗**：[V14 (temperature 從未 fire)](/posts/2026-08-11-arc3-v14-temperature-patch-failed/)、[V31 (7 modules 從未執行)](/posts/2026-08-30-arc3-v34-nooa-v3/)

**得來不易的校準點**：[V10 (純 baseline replica)](/posts/2026-08-09-arc3-v10-rokaiya-baseline/)

## 作者

Vincent — ocean240812@gmail.com — [Kaggle profile](https://www.kaggle.com/ocean240812) — ARC-AGI-3 Rank 59 / 2708 teams
