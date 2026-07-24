# prompt-library/

Accumulated generation prompts. **SYSTEM GENERATED** — this is agent output, not a place you author by hand under normal use.

## Layout

```
prompt-library/
└── {board}/
    └── {subject}/
        ├── base_prompt.md              ← subject-level prompt
        └── {grade}/
            └── prompt.md               ← grade-specific prompt
```

Literal example:

```
prompt-library/EXAMPLE_BOARD/EXAMPLE_SUBJECT/base_prompt.md
prompt-library/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/prompt.md
```

Grade folders use `Grade` + space + number. Board and subject folder names are the exact CLI strings.

Both files are plain Markdown — free-form generation guidance, no required schema, no frontmatter.

## Resolution order

`kb_access.load_prompt(board, subject, grade)` picks the most specific prompt that exists on disk:

1. `prompt-library/{board}/{subject}/{grade}/prompt.md` → `input_type = "grade_prompt"`
2. `prompt-library/{board}/{subject}/base_prompt.md` → `input_type = "base_prompt"`
3. `rulesets/universal_rules.md` → `input_type = "cold_start"`

So an empty `prompt-library/` is a valid state: the first run for a board/subject falls through to cold start. That is the intended first-run path.

`kb_access.load_prompt_at_level(...)` bypasses the order and loads exactly one level. The orchestrator uses it after a Check-2-only revision, so a regeneration uses the level that was just revised even when a more specific (possibly stale) file also exists.

## Read by

- **Generator Agent** (`agents/generator.py`) — the only reader. Calls `load_prompt()` to obtain its generation guidance.
- The orchestrator also calls `load_prompt()` / `load_prompt_at_level()` directly to track which level the current attempt is operating at, which is what decides whether a revision targets `base_prompt` or `grade_prompt`.

## Written by

- **Revision Agent** (`agents/revision.py`) produces the revised prompt text. It has two modes, `subject` and `grade`, corresponding to the two file locations.
- Persistence goes through `kb_access.save_prompt(board, subject, content, mode, grade=None)`, where `mode` is `"base_prompt"` or `"grade_prompt"`. `grade` is required for `"grade_prompt"`.

Writes are whole-file overwrites, not appends.

Note on the current code: `save_prompt` is the sole writer of this directory, and in the repo as it stands the live `orchestrator.py` path does not call it — revised prompts are carried in memory for the duration of a run. So expect this directory to stay empty until prompt persistence is wired into the orchestrator. Verify against `q-matrix-agents` for the version you are running.

## Seeding by hand

Supported but not the norm. `load_prompt` only checks file existence, so a hand-written `base_prompt.md` is picked up immediately and skips cold start for that subject. Useful when you already know what good generation guidance looks like for a board.

If you do this, expect it to be overwritten the first time the Revision Agent persists a prompt at the same level. Keep an authored copy elsewhere.
