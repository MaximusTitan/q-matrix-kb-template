# rulesets/

Evaluation rules. This is what "correct" means to the Eval and Judge agents.

Two levels: one universal file you maintain by hand, and per-grade files the system appends to.

## Layout

```
rulesets/
├── universal_rules.md                  ← YOU MAINTAIN. Seeded in this repo.
└── {board}/
    └── {subject}/
        └── {grade}/
            └── rules.md                ← SYSTEM GENERATED (appended)
```

Literal example:

```
rulesets/universal_rules.md
rulesets/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/rules.md
```

Grade folders use `Grade` + space + number (`Grade 8`). Board and subject folder names are the exact strings passed on the CLI.

## universal_rules.md — YOU MAINTAIN

Markdown. Applies to every board, subject, grade, and chapter. A copy is shipped in this repo as a working starting point; it is our own authored content, not board material, so it is safe to edit, extend, or replace wholesale for your context.

Three consumers, all in `q-matrix-agents`:

- **Eval Agent** (`agents/eval.py`) — reads it via `kb_access.load_rules()` as the rule set every generated CSV is checked against (Check 1).
- **Judge Agent** (`agents/judge.py`) — same, via `load_rules()`.
- **Generator Agent** (`agents/generator.py`) — reads it via `kb_access.load_prompt()` as the **cold-start** generation input, used only when no `prompt-library` prompt exists yet for the board/subject/grade. This is why it must read as usable generation guidance, not only as a checklist.

Never delete this file. `load_prompt` and `load_rules` both raise `FileNotFoundError` if it is missing, and there is no fallback below it.

Keep rule IDs stable when you edit. Downstream feedback messages cite them by ID.

## {board}/{subject}/{grade}/rules.md — SYSTEM GENERATED

Markdown, a flat list of `- ` bullets. Grade-specific rules that **extend** the universal rules; they never replace them. `load_rules()` concatenates universal rules, then a `## Grade-Specific Rules` heading, then this file's contents.

Written by the orchestrator when a human rejects a CSV that passed automated eval:

```bash
python orchestrator.py --reject \
    --board EXAMPLE_BOARD --subject EXAMPLE_SUBJECT \
    --grade "Grade 8" --chapter Chapter01_Example_Chapter_Title \
    --reason "Max 3 skills per concept"
```

That appends `--reason` verbatim as one bullet, via `kb_access.append_grade_rule()`. The file is created on first append.

Read by the Eval and Judge agents, through `load_rules()`. Nothing else reads it.

Hand-editing is safe here — the writer only ever appends, so your edits survive. Keep bullets short and testable; each one becomes a criterion an LLM evaluator applies literally.
