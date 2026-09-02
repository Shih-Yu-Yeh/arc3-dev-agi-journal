+++
title = "ARC3 Day 0: Stub Baseline - Kaggle Code Competition Rerun Pipeline First Contact"
date = 2026-07-31T08:03:00+08:00
draft = false
tags = ["ARC3", "ARC-AGI-3", "Kaggle", "pipeline"]
categories = ["ARC3 Dev Journal"]
summary = "First submission. Stub baseline with file_name param. Failed with ERROR. Learned the commit run vs submission rerun distinction."
lb_score = "ERROR"
version = "Stub"
status = "ERROR"
+++

## TL;DR

First submission. Stub baseline with file_name param. Failed with ERROR. Learned the commit run vs submission rerun distinction.

## Context

ARC-AGI-3 is a Kaggle code competition, not a research competition. The submission pipeline has two stages: (1) a commit run that runs the notebook end-to-end on the public 25 games and produces submission.parquet; (2) a hidden rerun where Kaggle's internal gateway serves 110 hidden games via `http://gateway:8001/api/games`.

This stub was meant to test pipeline mechanics: kernel push, commit run, submission, LB appearance. I expected it to fail; the question was where.

## Technical Choice

Used the simplest possible submission.parquet format: a single row with `file_name` parameter set. No solver, no vLLM, no model. Just a Python cell that writes one parquet row.

The choice was deliberate: minimize moving parts to isolate pipeline issues from solver issues.

## Parameter Decisions

| Parameter | Value | Rationale |
|---|---|---|
| submission format | 1 row, file_name param | Minimal test |
| solver | none | Isolate pipeline from solver |
| model | none | Skip vLLM startup |
| GPU | T4 (default) | Cheapest available |

## Local vs LB Score

- Local mean: n/a (no solver)
- Baseline (previous version): n/a
- Local delta: n/a
- LB score: **ERROR**

## Patch Verification

| Patch | Fired? | Marker |
|---|---|---|
| (none - stub) | n/a | n/a |

## Outcome Analysis

The submission returned ERROR status. Root cause: the `file_name` parameter was not a valid submission column for ARC-AGI-3. The competition expects `row_id`, `game_id`, `end_of_game`, `score` columns.

Two learnings from this failure:

First, the Kaggle code competition rerun pipeline is strict about submission.parquet schema. A 1-row stub works for research competitions but not for code competitions with hidden rerun.

Second, the commit run vs submission rerun distinction matters. The commit run completed successfully (the notebook ran end-to-end and wrote submission.parquet). The submission rerun then failed because the schema was wrong. These are two separate failure modes that require two separate checks.

The next submission (V2) would use the correct schema and a real solver.

## Next Version Plan

Switch to the Tufa Labs duck harness fork. Use the correct submission.parquet schema (`row_id`, `game_id`, `end_of_game`, `score`). Expect LB 0.5-1.5 based on Tufa Labs' self-reported variance.
