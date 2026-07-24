# curriculum-docs/

Board-level curriculum and learning-outcome documents. **YOU POPULATE THIS.** Nothing in the pipeline writes here — these are inputs only.

This is the primary generation input. Without a document here for a board/subject/grade, the Generator Agent cannot run for that combination.

## Layout

```
curriculum-docs/
└── {board}/
    └── {subject}/
        └── {grade}/
            └── (your .pdf / .md / .txt files)
```

Literal example:

```
curriculum-docs/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/learning_outcomes.pdf
```

Grade folders use `Grade` + space + number (`Grade 8`). Board and subject folder names are the exact strings passed on the CLI.

There is no per-chapter subdivision here. Curriculum docs are scoped to a grade, and every chapter in that grade reads the same set. Chapter-specific material goes in `textbooks/` instead.

## File formats

`kb_access.load_curriculum_docs()` walks the grade folder and concatenates, joined by `\n\n---\n\n`:

| Extension | Handling |
|---|---|
| `.md` | read as text |
| `.txt` | read as text |
| `.pdf` | text extracted at read time via `skills/pdf_reader.py` |

Any other extension is ignored silently. Filenames are not significant; order is filesystem order, so do not encode meaning in it.

Multiple files per grade folder are fine and are all loaded. Put the syllabus, the LO document, and any board circular in the same folder if you have them.

`load_curriculum_docs` raises `FileNotFoundError` if the folder is missing **and also** if the folder exists but yields no readable content — so an empty folder is not a soft-pass.

PDFs are routed through Git LFS by `.gitattributes`. Run `git lfs install` before adding the first one.

## Read by

- **Generator Agent** (`agents/generator.py`) — calls `load_curriculum_docs(board, subject, grade)` on every generation attempt.

Nothing else reads this directory. Nothing writes it.

## Rights

You must have the right to use, and — if this repo is public or shared — to redistribute anything you put here. Board syllabus and LO documents are frequently free to download and still fully copyrighted with no redistribution license. Freely accessible is not freely redistributable. If yours is accessible-but-not-redistributable, keep your clone private or leave these files untracked; the pipeline works either way.
