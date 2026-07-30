# q-matrix-kb-template

> Empty knowledge-base skeleton for the Q-Matrix curriculum generation system — pairs with [q-matrix-agents](https://github.com/MaximusTitan/q-matrix-agents).

This repository is the **structure** of the Q-Matrix data layer, with none of the data. It contains directories, `.gitkeep` files, per-directory `README.md` files, a set of `_PLACEHOLDER_README.md` notes, and one seeded ruleset (`rulesets/universal_rules.md`). It contains no code, no secrets, and **no curriculum material of any kind**.

**This repo ships empty on purpose.** No textbook, no syllabus document, no generated CSV. Every directory carries an `EXAMPLE_BOARD/EXAMPLE_SUBJECT/Grade 8/` tree, but those folders are a *shape demo* — deliberately fake names holding nothing but `.gitkeep` and a placeholder note, so the layout is unmistakable on first clone. You delete them (see [Quickstart](#quickstart) step 2). Q-Matrix reads curriculum material that you supply, and curriculum material is almost always someone else's copyright. Shipping a "starter dataset" would mean redistributing it. So the first-run experience is: clone this, then put your own material in.

---

## Using this repo, and where to contribute

**This is a template — cloning or forking it to build your own knowledge base is exactly what it is for.** That is the intended path, not a workaround. Take it, rename it, add directories, strip the parts you don't need.

It is open source and free to use, fork, and modify under its license (see [License](#license)). It is **not currently accepting pull requests**. That is a maintenance decision, not a closed door: the layout documented here is owned by `skills/kb_access.py` in [q-matrix-agents](https://github.com/MaximusTitan/q-matrix-agents), so structural changes start there and land here afterwards — never the other way round.

**If you want to contribute to Q-Matrix, [q-matrix-dataset](https://github.com/MaximusTitan/q-matrix-dataset) is the repo that is open for it.** Curriculum data contributions belong there, under its rights and attestation rules. Issues on this repo are open if you spot something wrong in these docs.

Forking the *structure* is free. What you put into your fork is a separate question — see the rights notice below.

## Paper

This repo is the data layer of the system described in *Curriculum Brain: A Semi-Automated Framework for Q-Matrix Creation* — [read the draft](https://prickly-gopher-95e.notion.site/Curriculum-Brain-3a3527ed7aee80cc97f7ee52e302249e).

## Paper

This repo implements the data schema and template described in *Curriculum Brain: A Semi-Automated Framework for Q-Matrix Creation* — [read the draft](https://prickly-gopher-95e.notion.site/Curriculum-Brain-3a3527ed7aee80cc97f7ee52e302249e).

---

## What Q-Matrix Does

Given curriculum documentation from an education board, Q-Matrix produces a validated curriculum CSV that maps:

```
Board → Subject → Grade → Chapter → Concept → Skill
```

plus prerequisite links at three levels (L1 within a chapter, L2 across chapters in a grade, L3 across earlier grades).

This knowledge base holds the inputs the agents read and accumulates the outputs they write.

---

## Rights notice — read before you add anything

You are responsible for the legal status of every file you put in your clone of this repo.

- Supply only curriculum material you **hold or have been granted the right to use** in this way — and, if your clone is public or shared, the right to **redistribute**.
- **"Freely accessible" is not "freely redistributable."** Official education-board material — syllabus documents, learning-outcome documents, prescribed textbooks — is very often downloadable at no cost from a government or board website and still under copyright, with no license permitting you to republish it. A public download link is not a redistribution grant.
- **Absence of a stated license is a "no", not a "maybe".** If you cannot find the actual license or terms covering a document, treat it as not redistributable. "It was on the internet for free" is not a license, and neither is "it's for education".
- If your material is accessible-but-not-redistributable: **keep your clone private, or keep those files untracked.** That is a fully supported way to run Q-Matrix. Nothing in the pipeline requires the KB to be public.
- Q-Matrix generates derived artifacts (concept-skill maps, CSVs, prompts) from your inputs. Derived output does not launder the rights of the input — check before you publish those either.

No file in this template repo is board or publisher material.

---

## How it pairs with q-matrix-agents

The code lives in `q-matrix-agents`. It resolves every KB path from a single environment variable, `KB_ROOT` (see `skills/kb_access.py` there — it is the only module that knows this folder layout).

```bash
# Clone the upstream skeleton — or swap in your own fork, which is the usual move.
git clone https://github.com/MaximusTitan/q-matrix-kb-template.git ~/q-matrix-kb
# then in the q-matrix-agents repo's .env:
KB_ROOT=/absolute/path/to/your/q-matrix-kb
```

Point `KB_ROOT` at your clone of **this** repo. Rename the clone to whatever you like — the name is not load-bearing, only the path in `KB_ROOT` is.

### Git LFS — set this up first

Chapter and curriculum PDFs are large binaries. `.gitattributes` in this repo already routes `*.pdf` through Git LFS. Install and initialize LFS **before** you add your first PDF — a PDF committed as a normal blob stays in history as a normal blob, and pulling it back out is a history rewrite.

```bash
git lfs install
```

---

## The five directories

```
q-matrix-kb-template/
│
├── rulesets/           ← eval rules. universal_rules.md is seeded; grade rules are generated.
├── prompt-library/     ← generation prompts. System-generated (seedable by hand).
├── curriculum-docs/    ← board syllabus / LO documents. YOU SUPPLY.
├── textbooks/          ← chapter PDFs (YOU SUPPLY) + all per-chapter generated output.
└── escalations/        ← dated failure snapshots. System-generated.
```

The agents also recognise a sixth top-level directory, `run_history/`, which
archives run records recovered from earlier git commits. It is deliberately absent
here: it is a recovery mechanism for a KB that already has history, not part of the
starting structure. `skills/kb_access.py` creates it on demand if it is ever needed.

| Directory | Populated by | Read by | Written by |
|---|---|---|---|
| `rulesets/universal_rules.md` | **You** (seeded here) | Eval, Judge, Generator (cold start) | Human, manually |
| `rulesets/{board}/{subject}/{grade}/rules.md` | System | Eval, Judge | `orchestrator.py --reject --reason ...` |
| `prompt-library/` | System | Generator | Revision Agent output, via `kb_access.save_prompt` |
| `curriculum-docs/` | **You** | Generator | Nothing — inputs only |
| `textbooks/.../chapter.pdf` | **You** | Map Extraction Agent | Nothing — inputs only |
| `textbooks/.../extraction_guidance.md` | System | Map Extraction Agent | `orchestrator.py --re-extract --map-guidance ...` |
| `textbooks/.../concept-skill-map.json` | System | Generator, Eval, Doctor | Map Extraction Agent |
| `textbooks/.../confirmed_curriculum.csv` | System | Prerequisite L2/L3, analytics | Orchestrator (final output) |
| `textbooks/.../run/{stage}/` | System | Dashboard / analytics | Orchestrator |
| `escalations/` | System | Dashboard / analytics | Orchestrator |

Each directory has its own `README.md` with the exact naming convention, file formats, and read/write ownership. Read those before populating.

---

## Naming conventions

These are enforced by convention, not by code validation. Get them wrong and the pipeline silently looks in the wrong place.

**Grades — `Grade` + a space + the number.**

```
Grade 8
Grade 10
```

The space is required. This is what the existing KB uses. `_grade_sort_key` in `kb_access.py` orders grades on the **trailing integer**, so a folder with no trailing digits (e.g. `Kindergarten`) sorts before everything and will not be treated as a later grade by the L3 prerequisite scan.

**Boards and subjects — the exact string you pass on the CLI.**

```
EXAMPLE_BOARD/EXAMPLE_SUBJECT/
CBSE/Science/
CBSE/Environmental Science/
```

Spaces are fine. Case matters. `--subject Science` will not find a folder named `science`.

**Chapters — `Chapter{N}_{Title_With_Underscores}`.**

```
Chapter04_Exploring_Forces
Chapter10_Sound
```

Zero-padding is optional and is inconsistent in real use; pick one style per subject and stay with it. **Avoid `:` `/` `\` `*` `?` `"` `<` `>` `|` in chapter names** — they are illegal in Windows filenames and will break the repo for anyone on Windows. Avoid trailing dots and spaces for the same reason.

The chapter folder name is also the `chapter` identifier written into every generated CSV. It does not need to match the chapter's printed title.

---

## Subject aliasing for L3 prerequisites

`_PREREQ_SUBJECT_ALIASES` in `kb_access.py` lets one subject borrow earlier grades from another when scanning for **L3 (cross-grade) prerequisites only**. As shipped:

```python
_PREREQ_SUBJECT_ALIASES = {
    "Science": ("Environmental Science",),
}
```

Rationale: CBSE starts `Science` as its own subject at Grade 6, while Grades 3–5 cover the same introductory ground as `Environmental Science`, so EVS grades are valid prerequisite sources for early Science grades.

This is a **configuration constant in code**, not a filesystem convention and not automatic behaviour. If your board splits or renames subjects across grades, edit that dict in `q-matrix-agents`. It does not affect L1 or L2, which stay exact-match on subject.

---

## Quickstart

```bash
# 0. Install Git LFS before any PDF exists in the repo.
git lfs install

# 1. Get the skeleton.
# Clone the upstream skeleton — or swap in your own fork, which is the usual move.
git clone https://github.com/MaximusTitan/q-matrix-kb-template.git ~/q-matrix-kb
cd ~/q-matrix-kb

# 2. Delete the shape-demo tree. It is documentation, not data.
rm -rf */EXAMPLE_BOARD

# 3. Supply YOUR curriculum material. Nothing works until this step is done.
#    Material you have the right to use — see the rights notice above.
mkdir -p "curriculum-docs/CBSE/Science/Grade 8"
cp ~/my-syllabus.pdf "curriculum-docs/CBSE/Science/Grade 8/"

mkdir -p "textbooks/CBSE/Science/Grade 8/Chapter10_Sound"
cp ~/my-chapter.pdf "textbooks/CBSE/Science/Grade 8/Chapter10_Sound/chapter.pdf"

# 4. Point the code at this clone.
#    In the q-matrix-agents repo, .env:
#      KB_ROOT=/Users/you/q-matrix-kb

# 5. Run the pipeline from q-matrix-agents.
python orchestrator.py --board CBSE --subject "Science" \
    --grade "Grade 8" --chapter "Chapter10_Sound"
```

Step 3 is the whole point. `rulesets/universal_rules.md` is the only content this repo gives you; everything else is yours to bring.

Outputs land back in this repo: `concept-skill-map.json` and `confirmed_curriculum.csv` in the chapter folder, run artifacts under `run/{stage}/`, prompts under `prompt-library/`, and a dated snapshot under `escalations/` if the run fails all its cycles.

---

## License

**Apache License 2.0**, Copyright 2026 Intelliana — see [LICENSE](./LICENSE). Same license as the sibling code repos, [q-matrix-agents](https://github.com/MaximusTitan/q-matrix-agents) and [q-matrix-graph-template](https://github.com/MaximusTitan/q-matrix-graph-template). ([q-matrix-dataset](https://github.com/MaximusTitan/q-matrix-dataset) is CC BY-SA 4.0 instead, because it ships data rather than code.)

The license covers the *skeleton* only — the directories, docs, and `rulesets/universal_rules.md`. It says nothing about curriculum material you add to your own clone; that keeps whatever rights its own rights holder gave it.

---

## Related

- **[q-matrix-agents](https://github.com/MaximusTitan/q-matrix-agents)** — orchestrator, agents, skills, dashboard (the code layer)
- **[q-matrix-dataset](https://github.com/MaximusTitan/q-matrix-dataset)** — the published curriculum dataset, and **the repo open for contributions**
- **[q-matrix-graph-template](https://github.com/MaximusTitan/q-matrix-graph-template)** — 3D knowledge-graph viewer for a curriculum export
- **[Curriculum Brain (paper draft)](https://prickly-gopher-95e.notion.site/Curriculum-Brain-3a3527ed7aee80cc97f7ee52e302249e)** — the framework these repos implement
