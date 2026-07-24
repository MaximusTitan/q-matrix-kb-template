# run_history/

Permanent archive of superseded run records. **SYSTEM GENERATED.** Read-only in practice; you should never write here by hand.

## Layout

```
run_history/
└── {board}/
    └── {subject}/
        └── {grade}/
            └── {chapter}/
                └── {run_id}.json
```

Literal example:

```
run_history/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/Chapter01_Example_Chapter_Title/<run_id>.json
```

Same naming conventions as the rest of the KB: `Grade` + space + number, exact CLI board/subject strings, `Chapter{N}_{Title_With_Underscores}`.

## Why it exists

`textbooks/.../run/{stage}/` holds only the **latest** run per stage. Before the per-stage layout existed, a single `run/` folder was overwritten by any later run, and re-runs destroyed earlier runs' cost and usage data.

This directory is where such records are archived when recovered — one file per run, keyed by the run's own `run_id`, so nothing here is ever overwritten. It mirrors the `escalations/` never-overwrite pattern, but for ordinary (non-escalated) runs.

## Files

Each `{run_id}.json` is one complete run record — the same shape as `run/{stage}/run.json` — written 2-space indented with `ensure_ascii=False` and insertion key order.

## Written by

- `kb_access.archive_run_history_record()`.

This is **not called by the live pipeline.** It backs a one-time recovery of runs reconstructed from Git history that predate the per-stage layout. In normal operation this directory stays as it is; expect it to be empty in a fresh KB, permanently.

## Read by

- `load_run_history_records(board, subject, grade, chapter)` — a chapter's archived runs, oldest storage first.
- `list_all_run_history_records()` — every archived record, found by walking this tree **directly** rather than by iterating `textbooks/` chapters. Deliberate: cost from a chapter since renamed or removed from `textbooks/` (a curriculum edit, not a re-run) still counts toward KB-wide totals.
- `load_all_run_records_for_chapter(...)` and `list_all_run_records()` — merge these records with live and escalated ones, deduped by `run_id`, with live records taking precedence.

Because records are keyed by `run_id` and located by folder path, a chapter folder renamed in `textbooks/` leaves its history here under the old name. Rename both, or accept the orphan — the KB-wide totals still include it either way.
