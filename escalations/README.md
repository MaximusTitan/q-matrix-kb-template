# escalations/

Human-in-the-loop failure snapshots. **SYSTEM GENERATED.** You never create these; you read them and act on them.

An escalation is written when a chapter's eval loop exhausts its attempts without producing a passing CSV. Each one is a self-contained snapshot of the run that failed — enough to diagnose it without reconstructing anything.

## Layout

```
escalations/
└── {board}/
    └── {subject}/
        └── {grade}/
            └── {chapter}/
                └── {YYYY-MM-DD}/
                    ├── report.md
                    ├── run.json
                    ├── attempt_{n}_prompt.md
                    └── *.csv                  (every artifact of the run)
```

Literal example:

```
escalations/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/Chapter01_Example_Chapter_Title/2026-01-01/report.md
```

The path mirrors `textbooks/{board}/{subject}/{grade}/{chapter}/` exactly, with a date folder appended. Same naming conventions: `Grade` + space + number, exact CLI board/subject strings, `Chapter{N}_{Title_With_Underscores}`.

Date folders are `YYYY-MM-DD`.

## Accumulation vs overwrite

Unlike `textbooks/.../run/{stage}/` — which holds only the latest run per stage — **escalation folders accumulate.** Each failure gets its own dated folder and is never overwritten. This directory is a chapter's failure history.

One consequence: two escalations for the same chapter on the same date land in the same folder, and the second overwrites the first's files.

## Files

| File | Contents |
|---|---|
| `report.md` | Human-readable summary. Header block of `**Field:** value` lines (`Board`, `Subject`, `Grade`, `Chapter`, `Date`, `Failed Check`, `Total Attempts`), then an `## Attempt History` section with a `### Attempt N (input_type: ...)` block per attempt carrying Check 1 / Check 2 status, feedback bullets, and missing concepts/skills. |
| `run.json` | The full structured run record — the same shape written to `run/{stage}/run.json`, including `run_id` and cost/usage. |
| `attempt_{n}_prompt.md`, `*.csv` | Every artifact of the escalated run, same filenames as in `run/{stage}/`. |

`report.md` is the marker file. `list_escalations()` returns **only** folders containing a `report.md`; empty escalation folders are ignored, so a stray directory here is harmless.

## Read by

- `kb_access.list_escalations()` — enumerates escalations for the dashboard. Identifiers come from parsing `report.md`'s header, not from the directory names, because the header preserves the original un-normalized `Grade`/`Chapter` spelling.
- `kb_access.load_escalation_report(folder)` — parses one escalation into header + per-attempt records + sibling filenames, and always returns `raw_report` as a complete fallback if parsing misses.
- `kb_access.list_all_run_records()` — pulls `run.json` from here too, so escalated runs count toward KB-wide cost totals. Deduped by `run_id`: a chapter currently in escalated state has the same run present in both `run/{stage}/` and here, and it is counted once.

Because header parsing is regex-based on `report.md`, **do not reformat or hand-edit `report.md`.** Breaking the `**Field:** value` header shape makes the escalation invisible to the dashboard's identifier columns. Delete a stale escalation folder outright rather than editing it — but note that also deletes that run's cost record unless it also exists in `run_history/`.

## Written by

- `kb_access.write_escalation()`, called by the orchestrator when the eval loop exhausts its attempts.
