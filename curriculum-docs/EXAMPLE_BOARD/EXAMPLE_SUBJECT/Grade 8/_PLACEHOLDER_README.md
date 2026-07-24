# Placeholder — delete me

Path: `curriculum-docs/EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/`

This is a **shape demo**, not data. These folders exist only so the directory layout
is unmistakable when you first clone the repo. `EXAMPLE_BOARD` and `EXAMPLE_SUBJECT`
are deliberately fake names — no real board or subject is named here, and there is no
curriculum content anywhere in this repo.

**Delete this whole tree before you use the repo:**

```bash
rm -rf */EXAMPLE_BOARD
```

Leaving it in place is harmless but noisy: `list_textbook_chapters()` and the dashboard
enumerate by walking these trees, so `EXAMPLE_BOARD` will show up as a real (empty)
board until you remove it.

Replace it with your own board, subject, grade, and chapter folders, following the
naming conventions in this directory's `README.md` and the root `README.md`:

- grade: `Grade` + a space + the number, e.g. `Grade 8`
- board / subject: the exact strings you pass on the CLI
- chapter: `Chapter{N}_{Title_With_Underscores}`, no Windows-illegal characters
