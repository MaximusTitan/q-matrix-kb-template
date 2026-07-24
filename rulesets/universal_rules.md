# Universal Rules — Curriculum CSV Quality Evaluation

**Applies to:** All boards, subjects, grades, and chapters.  
**Schema:** Board | Subject | Grade | Chapter | Concept | Skill (6 columns, no others)  
**Override:** Specialized rulesets at `rulesets/{board}/{subject}/{grade}/rules.md` apply on top of these rules, never instead of them.  
**Living document:** These rules will be updated as edge cases emerge. Rule IDs are stable — do not renumber; append new rules at the end of each section.

---

## Section 1 — Structural Checks

These checks are binary pass/fail. A single failure in this section blocks output entirely.

**R-S1 — Column count**  
Every row must have exactly 6 columns: Board, Subject, Grade, Chapter, Concept, Skill. No extra columns. No merged or split columns.

**R-S2 — Column names**  
The header row must contain exactly these six column names: board, subject, grade, chapter, concept, skill. 
Lowercase is the system standard. Any case variant is acceptable as long as the names are correct.

**R-S3 — No empty required fields**  
No cell in any of the 6 columns may be blank, null, or whitespace-only. Every row must be fully populated.

**R-S4 — Board consistency**  
All rows within a single generation run must have the same Board value. The Board value must match the board specified by the user input exactly (e.g., "CBSE", not "cbse" or "C.B.S.E.").

**R-S5 — Subject consistency**  
All rows within a single generation run must have the same Subject value.

**R-S6 — Grade consistency**  
All rows within a single generation run must have the same Grade value, formatted as "Grade N" (e.g., "Grade 9", not "9" or "Class 9" or "IX").

**R-S7 — No duplicate rows**  
No two rows may be identical across all 6 columns. A Chapter+Concept+Skill combination that appears more than once is a duplicate and must be removed.

**R-S8 — Chapter value fidelity**  
The Chapter value must match exactly the chapter identifier provided as input to the generation run (e.g., "Chapter10_Sound"). 
This is a system identifier and is not required to match the chapter title verbatim from the source document.

---

## Section 2 — Concept Quality

**R-C1 — Grounded in source document**  
Every Concept must correspond to a topic, subtopic, or learning area that is explicitly or clearly implicitly present in the source curriculum document. Concepts must not be invented, inferred from general subject knowledge, or imported from other chapters/grades.

_Failing example:_ Listing "Quantum Mechanics" as a concept under CBSE Grade 9 Science Chapter "Matter in Our Surroundings" — this topic is not part of the source.  
_Passing example:_ "States of Matter", "Effect of Temperature on States", "Evaporation and Factors Affecting It" — all grounded in the chapter.

**R-C2 — Appropriate granularity**  
Concepts must be at the right level — neither too broad nor too narrow. 
Granularity decisions must be based on how the source document treats the topic — if the source document gives separate headings or separate learning objectives to two related topics, they are distinct concepts regardless of their scientific relationship.

**R-C3 — Minimum concepts per chapter**  
Every chapter must have at least 2 distinct Concepts. A chapter that maps to only 1 concept is almost always too coarse — it means the chapter content was collapsed rather than unpacked.

**R-C4 — Concepts are not skills**  
A Concept must be a noun phrase describing a topic or idea, not a verb-led action statement. If a Concept row reads "Identify the states of matter", it is a skill written in the wrong column. Concepts describe what is being learned; skills describe what the student can do.

_Failing:_ Concept = "Calculate speed using distance and time"  
_Passing:_ "Acceleration", "Uniform vs Non-Uniform Motion", "Speed and Its Measurement" — each is a noun phrase scoped to one teachable idea

**R-C5 — No cross-chapter contamination**  
A Concept must belong to the Chapter it is listed under. Do not list a concept from Chapter 2 under Chapter 1 because it "relates" to the content there.

---

## Section 3 — Skill Quality

**R-SK1 — Verb-led phrasing**  
Every Skill must begin with an observable action verb (Bloom's Taxonomy or equivalent). The verb must be the first word of the skill statement.  
_Acceptable verb categories:_

- Remember: Define, List, Recall, Name, State
- Understand: Explain, Describe, Distinguish, Classify, Summarize, Identify
- Apply: Calculate, Solve, Use, Apply, Demonstrate, Convert
- Analyze: Compare, Differentiate, Analyze, Examine, Infer
- Evaluate: Justify, Evaluate, Assess, Critique
- Create: Design, Construct, Formulate, Propose

_Failing:_ "The student will know about photosynthesis" / "Understanding of motion" / "Cells and their types"  
_Passing:_ "Explain the process of photosynthesis using a labelled diagram" / "Differentiate between uniform and non-uniform motion with examples"

**R-SK2 — Demonstrates mastery of its Concept**  
Each Skill must be directly and specifically tied to its Concept. A skill that is merely "about the same chapter" but does not test the concept it is listed under is a misalignment.  
_Failing:_ Concept = "Evaporation", Skill = "Define the water cycle" (water cycle is a different concept)  
_Passing:_ Concept = "Evaporation", Skill = "Explain at least three factors that affect the rate of evaporation"

**R-SK3 — Student capability phrasing**  
Skills must describe what a student can do, not what a teacher does or what a lesson covers. Avoid phrasing like "Teach students to...", "Cover the topic of...", "Introduce...".

**R-SK4 — Count of skills per concept**  
R-SK4 — Concepts with exactly 1 skill are flagged for human review. Concepts with 8+ skills are flagged for human review. 
Counts of 2–7 skills per concept are all acceptable and must NOT be flagged.

**R-SK5 — Skills are not re-statements of the concept**  
A skill must add specificity beyond the concept name. If the Concept is "Newton's First Law of Motion" and the Skill is "Understand Newton's First Law of Motion", the skill adds nothing.  
_Failing:_ Concept = "Chemical Reactions", Skill = "Learn about chemical reactions"  
_Passing:_ Concept = "Chemical Reactions", Skill = "Identify the reactants and products in a given chemical equation"

**R-SK6 — No duplicate skills under the same concept**
Two skills are duplicates only if a student demonstrating one skill 
would automatically demonstrate the other. Skills that test different 
cognitive levels of the same concept are NOT duplicates even if they 
share vocabulary.

NOT duplicates (acceptable):
  - "Define frequency as vibrations per second"
  - "Relate frequency to the pitch of sound"
  These test different cognitive levels: recall vs. application.

DUPLICATES (flag these):
  - "List the states of matter"
  - "Name the three states of matter"
  These are identical tasks with different surface wording.

**R-SK7 — Skills do not re-use the concept verb wholesale**  
If the concept is "Photosynthesis" and every skill begins with "Define photosynthesis", "Explain photosynthesis", "Describe photosynthesis" — this is skill inflation with no real differentiation. Skills must test different facets or cognitive levels of the concept.

---

## Section 4 — Coverage Completeness

**R-CV1 — All major learning objectives represented**  
Every significant learning objective (LO) or learning area stated or clearly implied in the source curriculum document must be represented by at least one Concept+Skill pair. A CSV that accurately represents half a chapter's content fails this rule.

_What counts as a major LO:_ Any topic given a heading, subheading, or explicit statement in the source document. Margin notes, example boxes, and "did you know" sidebars do not count unless the source document explicitly frames them as learning objectives.

**R-CV2 — No fabricated coverage**  
Coverage must not be padded with invented topics to meet a count target. If a chapter has 4 real learning areas, the CSV should have 4 concepts — not 7, where the extra 3 are plausible-sounding but not in the source.

**R-CV3 — Chapter balance check (flag, not auto-fail)**  
If one chapter has significantly more concepts than another chapter of similar scope in the same subject and grade, flag for human review. This may be correct (some chapters are genuinely denser) but warrants a check.

**R-CV4 — No concept from a different chapter posing as coverage**  
Coverage cannot be achieved by importing concepts from other chapters. See also R-C5.

---

## Section 5 — Prerequisite Integrity

_Note: The current schema does not have a dedicated prerequisite column. These rules apply if Skill statements reference prerequisite knowledge, or if a future schema version adds prerequisite fields. They are included now so the eval layer can enforce them when prerequisites appear in skill phrasing._

**R-P1 — Prerequisite concepts must exist in scope**  
If a Skill statement references knowledge from another concept (e.g., "Using knowledge of cell structure, explain how diffusion occurs"), that referenced concept (cell structure) must either:  
(a) appear as a Concept in the same CSV, or  
(b) be a clearly established prerequisite from a prior grade — in which case the skill statement should acknowledge this (e.g., "Building on Grade 8 knowledge of...").

**R-P2 — Skills must not assume concepts not yet taught**  
Within a single grade+subject CSV, if Skill B requires understanding Concept A, Concept A must appear in the CSV. The CSV must be internally consistent — a skill cannot require knowledge the CSV itself does not provide.

**R-P3 — No circular prerequisite chains**  
Concept A cannot require Concept B as a prerequisite if Concept B also requires Concept A.

---

## Section 6 — Output Format

**R-F1 — CSV encoding**  
Output must be valid UTF-8 CSV. Fields containing commas must be wrapped in double quotes. Newlines within a field are not permitted.

**R-F2 — Header row required**  
The first row must be the header: `Board,Subject,Grade,Chapter,Concept,Skill`

**R-F3 — No trailing whitespace**  
No leading or trailing whitespace in any cell. " Grade 9 " is not the same as "Grade 9".

**R-F4 — Consistent chapter ordering**  
Rows should be grouped by Chapter in the order chapters appear in the source document. Within a chapter, rows should be grouped by Concept. Within a concept, skills should be ordered from lower to higher cognitive complexity where possible (e.g., Define before Analyze).

---

## Appendix A — Rule Classification

|Code|Type|Auto-Fail?|Notes|
|---|---|---|---|
|R-S1 to R-S8|Structural|Yes — all|Binary checks|
|R-C1|Concept|Yes|Core grounding rule|
|R-C2|Concept|Yes|Granularity|
|R-C3|Concept|Yes|Min 2 per chapter|
|R-C4|Concept|Yes|Not a skill|
|R-C5|Concept|Yes|No cross-chapter drift|
|R-SK1|Skill|Yes|Verb-led|
|R-SK2|Skill|Yes|Tied to concept|
|R-SK3|Skill|Yes|Student phrasing|
|R-SK4|Skill|Flag|Human review for 1-skill or 8+ skill concepts|
|R-SK5|Skill|Yes|Adds specificity|
|R-SK6|Skill|Yes|No semantic duplicates|
|R-SK7|Skill|Flag|Human review|
|R-CV1|Coverage|Yes|Major LOs represented|
|R-CV2|Coverage|Yes|No fabrication|
|R-CV3|Coverage|Flag|Balance check|
|R-CV4|Coverage|Yes|No cross-chapter import|
|R-P1 to R-P3|Prerequisite|Yes (if applicable)|Only triggered when prereqs appear in skill text or future schema adds prereq column|
|R-F1 to R-F4|Format|Yes — all|Output hygiene|

---

## Appendix B — Grounding Examples (CBSE Grade 9 Science)

These examples are drawn from the CBSE Grade 9 Science teacher resource document and serve as calibration anchors for what "good" looks like in this specific combination. They are illustrative, not exhaustive.

**Example: Chapter "Matter in Our Surroundings"**

|Chapter|Concept|Skill|
|---|---|---|
|Matter in Our Surroundings|States of Matter|Classify a given substance as solid, liquid, or gas based on its observable properties|
|Matter in Our Surroundings|States of Matter|Explain why gases are highly compressible while solids are not, using particle theory|
|Matter in Our Surroundings|Interconversion of States|Describe the effect of temperature on the state of matter with suitable examples|
|Matter in Our Surroundings|Interconversion of States|Distinguish between melting point and boiling point with reference to a specific substance|
|Matter in Our Surroundings|Evaporation|Identify at least three factors that affect the rate of evaporation|
|Matter in Our Surroundings|Evaporation|Explain why evaporation causes cooling using the concept of latent heat|

_What this example demonstrates:_

- Concepts are distinct sub-topics within the chapter (R-C2 ✓)
- Each concept has ≥2 skills (R-SK4 ✓)
- Skills are verb-led and specific (R-SK1, R-SK5 ✓)
- Skills test different facets of the concept (R-SK7 ✓)
- No skill re-states the concept name (R-SK5 ✓)

**What a failing version looks like (same chapter):**

|Chapter|Concept|Skill|
|---|---|---|
|Matter in Our Surroundings|Matter in Our Surroundings|Understand matter|
|Matter in Our Surroundings|Properties of Matter|Know about properties|
|Matter in Our Surroundings|Quantum Behaviour of Particles|Explain quantum tunnelling|

---

## Appendix C — How the Eval Layer Uses These Rules

1. **Structural rules (R-S) run first.** If any fail, generation is immediately flagged for regeneration — there is no point checking content quality on a malformed CSV.
    
2. **Content rules (R-C, R-SK, R-CV, R-P) run after structure passes.** Each rule that fails must produce a specific, actionable failure message: which rule, which row(s), what is wrong, what needs to change.
    
3. **Format rules (R-F) run last.** They are cheap and fast.
    
4. **Flag rules** (R-CV3, R-SK7) do not trigger regeneration. They are appended to the output as human review notes.
    
5. **Feedback structure for regeneration:**  
    When rules fail and a new generation is triggered, the feedback passed to the prompt must include:
    
    - The rule ID that failed
    - The exact rows that failed (Board/Subject/Grade/Chapter/Concept/Skill values)
    - A plain-English description of what is wrong
    - A plain-English instruction for what the corrected version should look like
6. **Maximum 3 iterations.** If the output still fails after 3 attempts, escalate to human with a failure report showing: which rules failed at each attempt, which rows were affected, and what changes were made between attempts.