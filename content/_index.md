+++
title = "ARC3 Dev Journal"
date = 2026-07-31T00:00:00+08:00
draft = false
+++

# ARC Prize 2026 (ARC-AGI-3) — Development Journal

33 days. 21 public leaderboard submissions. 33 private experiment versions. One question: can a single RTX Pro 6000 with Qwen3.8-27B-FP8 reach LB 2.0+ on 110 hidden interactive reasoning games?

The answer turned out to be yes — but not in the direction I expected.

## Headline Result

**V24 reached LB 2.56 on 2026-08-25** by upgrading `MULTIMODAL_UPSCALE` from 4 to 8 (256x256 to 512x512 vision rendering). Every other change — modules, grafts, prompt addenda, context budget, temperature tuning — either regressed or had no measurable effect.

## What This Journal Is

Each post in this journal documents one leaderboard submission. The structure is consistent:

- **Context** — why this direction, what hypothesis motivated it
- **Technical choice** — what was changed and why this approach over alternatives
- **Parameter decision** — explicit table of parameters, previous vs current, with rationale
- **Local vs LB score** — the gap between local mean and leaderboard, with diagnosis
- **Patch verification** — a 4-layer check (syntax, marker print, vLLM config, events.jsonl)
- **Outcome analysis** — what happened, why, what to do next

This is not a tutorial. It is a logbook of decisions, evidence, and corrections.

## Why Public

I made three expensive mistakes that each cost a full 30-hour cycle:

1. **V31**: claimed 7 modules installed. Actually 0 fired. A bare `except Exception: pass` hid an `AttributeError` for the entire 9-hour run.
2. **V35.1**: 2443 `UnboundLocalError` exceptions in step_env. The kernel still completed and submitted, but every step crashed silently.
3. **V25**: added 7 grafts on top of V24's winning config. Lost four critical patches (FP8 KV, writable bundle copy, reasoning effort cap, noop guard) in the process. LB dropped 2.56 to 1.42.

The pattern: regressions hide in the gap between "what the code claims" and "what the runtime actually did." This journal is an attempt to make that gap visible — for myself on future competitions, and for anyone else building AI agent submissions for Kaggle code competitions.

## Start Here

- [LB Progress chart and table](/lb-progress/) — the full 21-submission history at a glance
- [Lessons Learned](/lessons-learned/) — 16-rule development constitution + 5 largest technical lessons
- [About](/about/) — author bio, tech stack, contact
- [Posts](/posts/) — one post per submission, in chronological order

## Quick Navigation by Outcome

**Start here**: [Prologue - Kaggle platform and ARC-AGI-3 introduction](/posts/2026-07-30-arc3-prologue-kaggle-and-arc-agi-3/)

**Best**: [V24 - LB 2.56 (winner)](/posts/2026-08-25-arc3-v24-mm-upscale8-winner/)

**Worst**: [V21 - LB 0.00 (dummy submission)](/posts/2026-08-22-arc3-v21-dummy-submission/)

**Silent failures**: [V14 (temperature never fired)](/posts/2026-08-11-arc3-v14-temperature-patch-failed/), [V31 (7 modules never executed)](/posts/2026-08-30-arc3-v34-nooa-v3/)

**Hard-won calibration**: [V10 (pure baseline replica)](/posts/2026-08-09-arc3-v10-rokaiya-baseline/)

## Author

Vincent — ocean240812@gmail.com — [Kaggle profile](https://www.kaggle.com/ocean240812) — Rank 59 of 2708 teams on ARC-AGI-3.
