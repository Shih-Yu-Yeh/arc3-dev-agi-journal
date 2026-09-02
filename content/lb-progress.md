+++
title = "LB Progress"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Leaderboard 進度 — 21 次提交 (2026-07-31 ~ 2026-09-01)

## 分數時間軸

![LB Score Timeline](/images/lb-progress.png)

## Patch 數量 vs LB Score

![Patch vs LB](/images/patch-vs-lb.png)

右圖提出一個反直觀論點：更多已驗證 fire 的 patch 不代表更高 LB score。V24 (9 patches) 拿到 2.56。V35.2 (10 patches) 拿到 1.59。重要的 patch 是 vision resolution 與 FP8 KV cache；module 風格的 instrumentation 一致地退步。

## 所有提交

| # | 日期 | 版本 | LB | Delta | Patches Fired | 狀態 |
|---|------|------|-----|-------|---------------|------|
| 1 | 2026-07-31 | Stub | ERROR | — | 0 | Pipeline 測試失敗 |
| 2 | 2026-08-01 | V2 | 0.87 | — | 3 (T, CW, VLLM) | Baseline |
| 3 | 2026-08-02 | V4 | 1.06 | +0.19 | 5 | Triple-patch |
| 4 | 2026-08-04 | V6 | 0.75 | -0.31 | 5 | Context budget 退步 |
| 5 | 2026-08-08 | V9 | 1.10 | +0.35 | 3 | yw8837 replica |
| 6 | 2026-08-09 | V10 | 0.79 | -0.31 | 3 | 純 baseline calibration |
| 7 | 2026-08-10 | V13 | 0.87 | +0.08 | 0 | Registration mode only |
| 8 | 2026-08-11 | V14 | ERROR | — | 3 | Temperature patch 從未 fire |
| 9 | 2026-08-15 | V15 | 0.80 | -0.07 | 3 | DVM fork |
| 10 | 2026-08-17 | V17 | 1.43 | +0.63 | 5 | Qwen3.8 突破 |
| 11 | 2026-08-18 | V19 | 1.17 | -0.26 | 9 | 全部 fire 但 LB 退步 |
| 12 | 2026-08-22 | V21 | 0.00 | -1.17 | 9 | Dummy submission 3411 bytes |
| 13 | 2026-08-23 | V21 (resubmit) | 0.00 | 0.00 | 9 | 同樣 dummy 問題 |
| 14 | 2026-08-24 | V23 | 1.53 | +1.53 | 8 | Anim bundle + Qwen3.8 |
| 15 | 2026-08-25 | **V24** | **2.56** | **+1.03** | 9 | **Winner: MULTIMODAL_UPSCALE=8** |
| 16 | 2026-08-26 | V25 | 1.42 | -1.14 | 6 | 7 grafts 但掉了 FP8/WBC/RE/NG |
| 17 | 2026-08-27 | V28 | 1.72 | +0.30 | 9 | Text-only ablation |
| 18 | 2026-08-29 | V33 | 1.51 | -0.21 | 10 | NOOA v2 退步 |
| 19 | 2026-08-30 | V34 | 2.24 | +0.73 | 10 | NOOA v3 恢復但仍低於 V24 |
| 20 | 2026-08-31 | V35.1 | 1.59 | -0.65 | 10 | 2443 UnboundLocalError |
| 21 | 2026-09-01 | V35.2 | 1.59 | 0.00 | 10 | Syntax 修復，modules 淨負面 |

**Patch 圖例**：T = temperature set，CW = context window set，VLLM = vLLM server started，FP8 = FP8 KV cache，MM8 = MULTIMODAL_UPSCALE=8，Q38 = Qwen3.8 model，WBC = writable bundle copy，RE = reasoning effort cap，NG = noop guard verified，NOOA = NOOA modules active，V35M = V35 6 modules instantiated。

## 關鍵觀察

### Local Mean vs LB Score

最大差距：V35.2 local mean 6.44 (比 V24 local 4.984 +29%)，但 LB 1.59 (比 V24 LB 2.56 -38%)。Modules 只對熟悉的公開遊戲有幫助；hidden games 懲罰 directive-driven 行為。

### V24 的差異

V24 是唯一同時擁有以下項目的提交：
- FP8 KV cache (在 vLLM 啟動指令中驗證：`--kv-cache-dtype fp8`)
- MULTIMODAL_UPSCALE=8 (驗證：`Patched MULTIMODAL_UPSCALE to 8`)
- Qwen3.8-27B-FP8 model (驗證：`Qwen/Qwen3.8-27B-FP8`)
- Writable bundle copy (避免 V11 的 read-only filesystem 錯誤)
- Reasoning effort cap 到 `high`
- Noop guard 驗證 (`hard_noop_guard = True`)
- Animation-awareness 開啟 (anim bundle)

V23 有以上全部，除了 `MULTIMODAL_UPSCALE=8`。結果：1.53。單一改動，+1.03 LB。

### V25 的錯誤

V25 在 V24 的 MM_UPSCALE=8 上加 7 個 grafts (winframe、goalkeep、clockwatch、hudmask、clickmap、searchmap、lawbook)。但 V25 實作退步了 V24 擁有的 4 個關鍵 patch：FP8 KV cache、writable bundle copy、reasoning effort cap、noop guard。LB 從 2.56 掉到 1.42。

教訓：加 feature 不是免費的，如果基礎退步了。

### V31 的幻影

V31 stdout 顯示 `v31: 7 modules installed:` 與 `v31: Ready. Expected: v24(2.56) + cross-game learning + efficiency = 3+`。但 events.jsonl 在 1348 個 events 中有 0 個提到 NOOA。一個 non-fatal `AttributeError: _system_prompt` 靜默地停用了每個 module。未提交到 LB。

### V35.1 / V35.2 的教訓

V35.1 有 2443 個 `UnboundLocalError: _v35_last_level` 例外在 step_env (closure scope bug)。LB 1.59。

V35.2 修了語法 (`nonlocal` 宣告)，0 個錯誤。LB 1.59。相同分數。

6 個 modules (ReasoningMemory、ExplorationTracker、HypothesisEngine、TransferMechanism、ReflectionRecovery) 在 V35.2 正確執行，但貢獻零淨值。這是 modules 對 ARC-AGI-3 是淨負面最強的證據。
