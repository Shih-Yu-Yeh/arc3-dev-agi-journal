+++
title = "Lessons Learned"
date = 2026-08-01T00:00:00+08:00
draft = false
+++

# Lessons Learned — 33 天，21 次提交，33 個私下版本

## 開發憲法 v2.0 (16 條)

活的文件，每次提交後更新。防止退步的規則粗體；從退步中學到的規則標記。

### 流程規則

**§1.** 本地 dry-run 是每次 push 前的強制步驟。無例外，連「一行修復」也算。

**§2.** 一個版本只改一個變數。V6 違反這條 (context budget + offline env_dir fix) 並退步 -0.31 LB。原因無法歸因。

**§3.** 永遠不寫 bare `except Exception: pass`。V31 的 `system prompt injection failed (non-fatal)` 靜默地停用了每個 module 整個 9 小時 run。

**§4.** 驗證 event 內容，不只驗 event count。V31 有 1348 個 events。沒有一個提到 NOOA。「modules installed」log 是假的。

**§5.** 每個 patch 必須 emit 一個 marker print。靜默套用的 patch 無法 debug。

**§6.** 同時跑 `kaggle kernels output` 與透過 `/api/v1/kernels/output` API 取 `kernel_log.json`。兩者涵蓋不同 artifacts。

### 驗證規則

**§7.** Patch 驗證是 4 層檢查：
1. Syntax：notebook source 含 patch code。
2. Semantic：stdout 含預期 marker print。
3. Behavioral：vLLM / solver 以 patched config 啟動。
4. Result：events.jsonl 顯示 patch 有效果。

V14 通過第 1-2 層但失敗第 3 層 (temperature 從未寫入)。V31 通過第 1-2 層但失敗第 4 層 (NOOA 從未執行)。

**§8.** Local mean 不是 LB score。永遠在結論前提交到 LB。

**§9.** 在公開遊戲上的 local mean 改善不會轉移到 hidden games。V35.2 local +29% vs V24，LB -38% vs V24。

### 技術規則

**§10.** 用 `frame[-1]`，不用 `frame[0]`。Film strip 的第一 frame 不是最終盤面狀態。在 `ls20` 上，它們差 4096 個 cell 中的 4012 個。(來源：busyaprime kernel)

**§11.** 不要信 `available_actions` 當 liveness signal。Engine 在遊戲結束後回空 frame，但 `available_actions` 維持全滿。5000 個 random steps 中有 2364 個是盲的。(來源：busyaprime kernel)

**§12.** 永遠傳 `x` 和 `y` 給 `ACTION6`。沒帶座標時，25 個遊戲中有 5 個會在遊戲 code 內 raise `KeyError`。(來源：busyaprime kernel)

**§13.** 不要假設 level order 等於 difficulty order。`baseline_actions` 只在 25 個遊戲中的 7 個在最後一個 level 達到峰值。(來源：busyaprime kernel)

**§14.** 用 writable bundle copy。`/kaggle/input/` 是 read-only。V11 死在 `OSError: [Errno 30] Read-only file system`。

### 資源規則

**§15.** 每週 GPU quota 規劃。每週 30 小時。V35 cycle 燒了 325 小時 wallclock (V24 的 60 小時)，因 modules 造成無限迴圈。

**§16.** Hidden games 是 air-gapped 的。沒有公開 API 暴露它們。學習 hidden game 行為的唯一方法是 instrumentation 自己的提交，用 level-probe graft。

---

## 5 個最大技術教訓

### 1. Local Mean 不是 LB Score

V35.2 local mean：6.44 (比 V24 local 4.984 +29%)。
V35.2 LB score：1.59 (比 V24 LB 2.56 -38%)。

改善 6 個熟悉公開遊戲效能的 modules 傷害了 85 個不熟悉 hidden games 的效能。Hidden game 分佈對 directive-driven 行為是對抗性的。

### 2. 每個 module 加法都退步 LB

| 版本 | Module 類型 | Local Δ | LB Δ vs V24 |
|------|-------------|---------|--------------|
| V25 | 7 TAAF grafts | n/a | -1.14 |
| V33 | NOOA v2 | n/a | -1.05 |
| V34 | NOOA v3 | n/a | -0.32 |
| V35.1 | 6 個 custom modules | n/a | -0.97 |
| V35.2 | 6 個 custom modules (修復) | +29% | -0.97 |

Tufa Labs 經驗上「hand-crafted tools degrade model performance」對 ARC-AGI-3 成立。

### 3. V31 幻影 Modules

V31 stdout 宣稱 `7 modules installed`。實際：0 個 modules 執行。

根因：`solver._system_prompt` 屬性在 `HarnessSolver` 上不存在。System prompt 注入 code 崩潰 raise `AttributeError`，被 bare `except Exception: pass` 接住。Modules 載入但沒有注入點。

偵測：grep events.jsonl 找「NOOA」— V31 有 0 個，V32 (修復) 有 469 個。

### 4. V35.1 UnboundLocalError x 2443

```python
def step_env(...):
    if some_condition:
        _v35_last_level = current_level  # 指派讓它變 local
    # ...後面...
    if _v35_last_level != current_level:  # branch 沒走時 UnboundLocalError
        ...
```

修法：在 `step_env` 頂部加 `nonlocal _v35_last_level`。

這個 bug 比邏輯 bug 難抓因為：
- Kernel 仍完成 (例外被接住)。
- stdout 是 624 KB (V24 的 134 KB)—滿是錯誤雜訊。
- LB 回真實分數 (1.59)，所以提交看起來「成功」。

偵測：grep stdout 找 `UnboundLocalError`。V35.1：2443 hits。V35.2：0 hits。

### 5. MULTIMODAL_UPSCALE=8 是決定性一擊

V23 (LB 1.53) 與 V24 (LB 2.56) 差一個改動：

```python
# V23
'MULTIMODAL_UPSCALE': '4',

# V24
'MULTIMODAL_UPSCALE': '8',
```

Local mean：V23 3.008，V24 4.984 (+66%)。LB：V23 1.53，V24 2.56 (+67%)。

對 ARC-AGI-3 來說，vision resolution 比 reasoning effort、context window size、module sophistication 都重要。512x512 grid rendering 保留了 256x256 丟失的 sub-cell pattern。

---

## 沒有效的方案

| 方案 | 版本 | 結果 | 診斷 |
|------|------|------|------|
| Context budget 32768 → 49152 | V6 | -0.31 LB | 更大 context 在小盤面上稀釋注意力 |
| Temperature 0.6 → 0.3 | V14 | ERROR | Patch 從未寫 env var；kernel 因其他原因 ERROR |
| 2-pass visible updates | V21 | 0.00 LB | submission.parquet 是 3411 byte dummy；pipeline 問題 |
| 在 V24 上加 7 TAAF grafts | V25 | -1.14 LB | 比 V24 掉了 FP8 KV、WBC、RE、NG |
| Text-only ablation | V28 | -0.84 LB | Hidden games 偏好 image modality |
| NOOA v2 (Memory + Supervisor) | V33 | -1.05 LB | Hysteresis 邏輯太激進地 redirect solver |
| NOOA v3 (V31 done right) | V34 | -0.32 LB | Wiring 修了；modules 仍淨負面 |
| 6 個 custom modules + closure bug | V35.1 | -0.97 LB | step_env 有 2443 個 UnboundLocalError |
| 6 個 custom modules + syntax 修復 | V35.2 | -0.97 LB | Modules 正確跑；仍淨負面 |

## 有效的方案

| 方案 | 版本 | 結果 | 為什麼 |
|------|------|------|--------|
| Tufa Labs duck harness fork | V2 | 0.87 LB | 穩固基礎；5 步循環健全 |
| Program synthesis prompt | V4 | +0.19 LB | 鼓勵結構化 tool call (但 V9 ablation 顯示其實 -0.04) |
| Qwen3.8-27B-FP8 升級 | V17 | +0.63 LB | 比 Qwen3.6 更好的 tool-call 準確度 |
| Anim bundle + wheelhouse 修復 | V23 | +1.53 LB | Animation-awareness 重要 |
| MULTIMODAL_UPSCALE 4 → 8 | V24 | +1.03 LB | Vision resolution 是瓶頸 |
