# textbooks/

Per-chapter input and per-chapter output, in the same folder. **MIXED OWNERSHIP** — you supply exactly one file per chapter; the system writes everything else.

This is the busiest directory in the KB and the only one that holds the final deliverable.

## Layout

```
textbooks/
└── {board}/
    └── {subject}/
        └── {grade}/
            └── {chapter}/
                ├── chapter.pdf                        ← YOU SUPPLY
                ├── extraction_guidance.md             ← SYSTEM (--re-extract)
                ├── concept-skill-map.json             ← SYSTEM (Map Extraction Agent)
                ├── confirmed_curriculum.csv           ← SYSTEM (FINAL OUTPUT)
                └── run/
                    └── {stage}/                       ← stage ∈ full | l1_prereq | l2 | l3
                        ├── run.json
                        ├── report.md
                        ├── gen_attempt_{n}.csv
                        ├── doctored_attempt_{n}.csv
                        ├── doctored_rules_attempt_{n}.csv
                        ├── attempt_{n}_prompt.md
                        └── confirmed.csv | last_csv.csv
```

Literal example:

```
textbooks/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/Chapter01_Example_Chapter_Title/chapter.pdf
textbooks/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/Chapter01_Example_Chapter_Title/confirmed_curriculum.csv
textbooks/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/Chapter01_Example_Chapter_Title/run/full/run.json
```

Grade folders use `Grade` + space + number. Board and subject folder names are the exact CLI strings. Chapter folders use `Chapter{N}_{Title_With_Underscores}` — see the root README for the full rule, including the Windows-illegal characters to avoid.

The chapter folder name **is** the `chapter` identifier that appears in every generated CSV row. Renaming a chapter folder after a run orphans that chapter's `escalations/` entries — they are keyed by name, not by ID.

This directory is also the KB's inventory: `list_textbook_chapters()` enumerates chapters by walking exactly this tree, and pipeline status is inferred from file presence, not from any manifest.

## chapter.pdf — YOU SUPPLY

The chapter's source PDF. The filename must be exactly `chapter.pdf`; `kb_access.get_chapter_pdf_path()` looks for that literal name and raises `FileNotFoundError` otherwise.

Read by the **Map Extraction Agent** (`agents/map_extraction.py`). Nothing writes it.

Routed through Git LFS by `.gitattributes`. Run `git lfs install` before adding the first one. Same rights rules as `curriculum-docs/` — see below.

## extraction_guidance.md — SYSTEM GENERATED

Markdown. A human instruction that constrains concept-skill map extraction for this one chapter, when the first extraction came out wrong.

Written by the orchestrator only under `--re-extract`:

```bash
python orchestrator.py --re-extract \
    --board EXAMPLE_BOARD --subject EXAMPLE_SUBJECT \
    --grade "Grade 8" --chapter Chapter01_Example_Chapter_Title \
    --map-guidance "Only NCERT LO-aligned concepts"
```

`--map-guidance` is required with `--re-extract`. The write is a whole-file overwrite (`kb_access.save_extraction_guidance`).

Read by the **Map Extraction Agent** via `load_extraction_guidance()`, which returns `None` when the file is absent — so its absence is the normal state, not an error. Once written, it applies to every subsequent extraction of that chapter until you delete it.

Hand-editing works and is a reasonable way to iterate, since the agent simply reads whatever text is there.

## concept-skill-map.json — SYSTEM GENERATED

JSON, written by the **Map Extraction Agent** (`kb_access.save_concept_skill_map`), 2-space indented, `ensure_ascii=False`.

```json
{
  "board": "EXAMPLE_BOARD",
  "subject": "EXAMPLE_SUBJECT",
  "grade": "Grade 8",
  "chapter": "Chapter01_Example_Chapter_Title",
  "concepts": ["<noun-phrase topic>", "<noun-phrase topic>"],
  "skills": ["<Verb-led statement>", "<Verb-led statement>"]
}
```

Concepts are noun phrases; skills are verb-led. Read by the Generator (as the coverage target), the Eval Agent (Check 2, CSM coverage), and the Doctor Agent (coverage repair). Overwritten on every re-extraction. Do not hand-edit — use `--re-extract` with `--map-guidance` instead, so the change is reproducible.

## confirmed_curriculum.csv — SYSTEM GENERATED, FINAL OUTPUT

**This is the deliverable, and it lives per chapter.** There is no aggregate CSV anywhere in the KB; a grade's curriculum is the union of its chapters' confirmed CSVs.

Written by the orchestrator via `kb_access.save_confirmed_csv()` when a chapter's CSV is confirmed. Overwrites unconditionally — the latest confirmation wins.

Base columns are `board, subject, grade, chapter, concept, skill`. Prerequisite runs append their own column sets, so the header grows as later stages complete: L1 (`prereq_*_L1_same_chapter`) after L1 mapping, then L2 (cross-chapter, same grade), then L3 (cross-grade, same subject). The exact column names are owned by `L1_COLUMNS` / `L2_COLUMNS` / `L3_COLUMNS` in `agents/prerequisite*.py`.

Because of that, **column presence and cell content mean different things**, and `kb_access` exposes both checks:

- `confirmed_csv_l2_attempted` / `confirmed_csv_l3_attempted` — did that stage run at all (are its columns in the header)?
- `confirmed_csv_has_prereqs` / `..._has_l2_prereqs` / `..._has_l3_prereqs` — did it actually find anything (is any cell non-empty)?

A confirmed CSV can exist with empty L1 columns: the prerequisite phase is defensive and still writes the checkpoint if mapping failed. "Confirmed CSV exists" does not mean "prerequisites were mapped."

Also read as input by later stages: L2 batch-loads every sibling chapter's confirmed CSV in the grade, and L3 batch-loads every chapter's confirmed CSV across all earlier grades (plus alias subjects — see the root README on `_PREREQ_SUBJECT_ALIASES`). L2 requires L1 complete for the grade; L3 requires L1 complete for every earlier grade. So a missing or unparseable confirmed CSV in one chapter degrades prerequisite quality for its siblings and successors — it is skipped, not raised.

## run/{stage}/ — SYSTEM GENERATED

Per-stage working record of the latest run. `{stage}` is derived from the run's `mode` (`_RUN_STAGE_SLUGS` in `kb_access.py`):

| mode | folder |
|---|---|
| `full` | `run/full/` |
| `prerequisite_only` | `run/l1_prereq/` |
| `l2_prerequisite_only` | `run/l2/` |
| `l3_prerequisite_only` | `run/l3/` |

An unrecognized mode falls back to a slug derived from the mode string, so forward-compatible stages get their own folder rather than colliding.

Stage separation is the point: `save_run_record()` **deletes and recreates only the one stage folder** it is writing. A later L2/L3 run therefore cannot destroy the full pipeline's cost and usage data, and a shorter re-run of a stage cannot leave orphaned siblings from a longer previous run of that same stage. Sibling stages are untouched.

Contents:

| File | Contents |
|---|---|
| `run.json` | structured source of truth; holds `run_id`, `mode`, cost/usage, and `*_file` pointers to the siblings |
| `report.md` | human-readable summary rendered from `run.json` |
| `gen_attempt_{n}.csv` | Generator output for attempt *n* |
| `doctored_attempt_{n}.csv` | Coverage Doctor output for attempt *n* |
| `doctored_rules_attempt_{n}.csv` | Rules Doctor output for attempt *n* |
| `attempt_{n}_prompt.md` | snapshot of the generation guidance used for attempt *n* |
| `confirmed.csv` or `last_csv.csv` | final CSV of the run — `confirmed.csv` on success, `last_csv.csv` otherwise |

Read back by `load_run_record()` and `load_run_artifact()`, which feed the dashboard and analytics. `load_run_artifact` enforces a strict filename whitelist (`^[A-Za-z0-9_]+\.(csv|md|json)$`) because the filename arrives from an API query parameter — so do not add files here whose names fall outside that alphabet; they will be unreadable through the API.

`run/` is a working directory, not an archive. Records that would be lost to overwrite are preserved in `escalations/` (failures, by date).

## Rights

`chapter.pdf` is the concern here. Supply only chapters you have the right to use and — if this repo is public or shared — to redistribute. Prescribed textbooks are usually free to download from a board website and still fully copyrighted, with no redistribution grant. Freely accessible is not freely redistributable. If yours is accessible-but-not-redistributable, keep your clone private or leave the PDFs untracked; the pipeline reads them from disk and does not care whether Git tracks them.

Generated artifacts in this folder are derived from your PDFs. Being derived does not by itself clear the rights on the source — check before publishing those too.
