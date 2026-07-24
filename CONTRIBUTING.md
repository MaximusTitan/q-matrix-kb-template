# Contributing to q-matrix-kb-template

This repo takes **curriculum-data** contributions only.

## Code changes go somewhere else

If your change is to the pipeline — a new agent, a prompt refinement, cost reduction, a new board adapter, a new prerequisite level, dashboard work, a bug in path resolution — open it against **[q-matrix-agents](https://github.com/MaximusTitan/q-matrix-agents)**, not here. This repo contains no code. `skills/kb_access.py` in that repo is the sole owner of the folder layout documented here; a structural change starts there and is reflected here afterwards, never the other way round.

Documentation fixes to this repo's own `README.md` files — a naming convention stated wrong, a path that no longer matches `kb_access.py` — are welcome here.

## Rights attestation

Every PR that adds or modifies curriculum material must state, in the PR description:

> I attest that I hold, or have been explicitly granted, the right to redistribute the curriculum material in this pull request under this repository's license, and that redistribution here does not breach any copyright, license, or terms of use governing it.

PRs adding curriculum material without this attestation are closed unmerged. This is not a formality — this repo is public, and a merge is a redistribution.

If you cannot make that statement truthfully, do not open the PR. See below.

## Accessible is not redistributable

This is the distinction that decides almost every case, and it is routinely got wrong.

**Freely accessible** means you can obtain a copy at no cost — a public download link, no login, no payment. Official education-board material is very often freely accessible: syllabus documents, learning-outcome frameworks, prescribed textbook PDFs, board circulars.

**Freely redistributable** means the rights holder has granted you permission to republish that copy elsewhere. That requires an actual grant: an explicit open license (Creative Commons, Open Government Licence, or similar), a written permission, or a public-domain status.

Accessible does not imply redistributable, and the overlap is small. A government or board website that offers a textbook PDF for free download has typically granted you nothing beyond personal or classroom use, and its terms-of-use page frequently says so outright. "It was on the internet for free" is not a license. Neither is "it's for education" — fair use and fair dealing are defences that turn on specific facts, and they do not clear a blanket republication in a public repo.

Before attesting, find the actual license or terms. If you cannot find one, treat the material as not redistributable. Absence of a stated license is a "no", not a "maybe".

Things that generally **are** contributable:

- Material under an explicit open license permitting redistribution — check the license text covers redistribution, not just access.
- Material you authored yourself, or your organization owns.
- Material you have written permission from the rights holder to redistribute.
- Genuinely public-domain material — confirmed, not assumed.
- Structural and documentation contributions: directory layout, `README.md` corrections, naming-convention clarifications.
- Rules and prompts you authored, e.g. additions to `rulesets/universal_rules.md`.

Things that generally are **not**:

- Prescribed textbook PDFs and chapter scans, whatever the source.
- Board syllabus and learning-outcome documents without an open license.
- Verbatim extracted text from any of the above.
- Concept-skill maps or CSVs that reproduce a copyrighted document closely enough to be a substitute for it. Being machine-generated does not clear the source's rights.

## If you only have accessible-but-not-redistributable material

**Keep it local. Do not open a PR.**

This is a fully supported way to run Q-Matrix, and it is what most users should do:

- Keep your KB clone **private**, or leave the material untracked (add the paths to your local `.gitignore`, or to `.git/info/exclude` so the ignore rule itself stays out of the repo).
- The pipeline reads from the filesystem via `KB_ROOT`. It never checks whether Git tracks a file. A completely untracked KB works identically to a committed one.
- If your team shares a KB, share it inside your organization on a private remote, where your internal use rights apply.

Nothing about Q-Matrix requires a public KB. This template is public so the *structure* is shareable; the data layer is expected to be private for most users.

## Practical notes for data PRs

- **Git LFS first.** `.gitattributes` routes `*.pdf` through LFS. Run `git lfs install` before adding a PDF. A PDF committed as a plain blob is in history for good short of a rewrite.
- **Naming conventions are not validated by code.** Follow them exactly: `Grade 8` (with the space), board/subject folders matching the CLI strings verbatim, chapters as `Chapter{N}_{Title_With_Underscores}` with no Windows-illegal characters (`: / \ * ? " < > |`, trailing dots, trailing spaces). See the root `README.md`.
- **Do not PR generated output.** `prompt-library/`, `concept-skill-map.json`, `confirmed_curriculum.csv`, `run/`, and `escalations/` are agent-written. They belong in your own KB clone, not in this template.
- **Do not include the `EXAMPLE_BOARD` trees in a data PR.** They are shape demos meant to be deleted downstream.
- **State the provenance** of every file you add: where it came from, and the specific license or permission that covers it. A URL is provenance; it is not a license.
