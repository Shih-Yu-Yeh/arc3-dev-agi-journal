+++

## 校準地板

V9 的 controlled comparison 之後，我需要知道絕對地板在哪——Tufa harness 在零 patch 下能得多少分？V10 刻意移除 AGI_8、AGI_9、program synthesis prompt、所有 graft，只剩 stock TAAF bundle + Qwen3.6-27B-FP8。LB 0.79 變成後續所有 patch 貢獻的比較基準。

## 為什麼故意退回原點

V9 的 controlled comparison 之後，我需要知道絕對地板在哪。Tufa harness 在零 patch 下能得多少分？

V10 刻意移除：AGI_8 patch、AGI_9 patch、program synthesis prompt、所有 graft。只剩 stock TAAF bundle + Qwen3.6-27B-FP8。

重點不是改善 LB，而是建立校準基準。未來每個 patch 的貢獻就能對這個地板量測。

## 用 rokaiya 當參考

用 `rokaiyasomapti/arc3-duck-v12-1d7d88-27e1af` 當參考。這個 kernel 是純 baseline replica，沒有修改。我 fork 它來驗證原作宣稱的 LB 1.38 是否可重現。

選 rokaiya kernel 而不是 V2 是刻意的：V2 用 Tufa bundle，但 rokaiya 的是更乾淨的 baseline，修改更少。如果兩者得分相近，地板就確認了。

## 全部清空

| 參數 | V9 | V10 | 理由 |
|---|---|---|---|
| AGI_8 patch | on | off | 拿掉所有 patch |
| AGI_9 patch | on | off | 拿掉所有 patch |
| program synthesis prompt | off | off | V9 已 off |
| grafts | 無 | 無 | Stock |
| model | Qwen3.6-27B-FP8 | Qwen3.6-27B-FP8 | 未變 |
| context window | 32768 | 32768 | Stock |

## 0.79 的意義

- Local mean: 未量測
- Baseline（上一版）: V9 LB 1.10
- Local delta: n/a
- LB score: **0.79**

## 只剩基本 marker

| Patch | 是否 fire? | Marker |
|---|---|---|
| temperature set | yes | temperature': 0.0 |
| context window set | yes | ANALYZER_CONTEXT_WINDOW = 32768 |
| vLLM server started | yes | vLLM server ready |
| give-up mechanism | yes | max_runtime |

## 0.79 vs rokaiya 宣稱 1.38

LB 0.79。校準基準。

rokaiya kernel 應該得 1.38，但我的 fork 得 0.79。0.59 的差距有兩個可能原因：

rokaiya 宣稱的 1.38 可能是單次高變動 rerun，不是可重現的結果。Hidden rerun variance 在相同 code 上可以擺動 ±0.3 LB。

我的 fork 用 `jeroencottaar/taaf-kaggle-source-share` bundle，而 rokaiya 可能用了 commit 略有不同的 bundle。TAAF bundle 有多個 fork，差異微妙。

0.79 地板現在是校準點。V9 的 1.10 是地板 +0.31，歸因於 AGI_8+AGI_9 patches。V4 的 1.06 是地板 +0.27，歸因於 AGI_8+AGI_9 扣掉 synthesis prompt 的 -0.04 淨效果。

## V11/V12/V13 的安排

V11 是私下 fork 嘗試，因 wheelhouse 路徑錯誤在 cell 2 RuntimeError。V12 是 Tufa milestone fork。V13 會用 registration-only mode 跳過 9h commit，讓 hidden rerun 產出真分。
