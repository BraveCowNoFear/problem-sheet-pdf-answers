---
name: problem-sheet-pdf-answers
description: Generate fully worked English answer PDFs for problem sheets, exercise sheets, homework PDFs, tutorial sheets, worksheets, or exam-style PDF handouts. Use when Codex receives a problem sheet PDF or problem images and the user asks for answers, solutions, worked solutions, a solution PDF, LaTeX-formatted answers, textbook-style explanations, diagrams, plots, or strict page-by-page visual and content QA with a final subagent review.
---

# Problem Sheet PDF Answers

## Non-Negotiables

- Produce a final PDF in English only.
- Use original problem order and answer every visible question, including subparts, footnotes, diagrams, tables, and hidden continuation text.
- Build the final PDF from real LaTeX source; do not assemble screenshots of text as the answer body.
- Include mathematical notation as LaTeX math, not plain-text approximations.
- Include relevant images or diagrams when they help the solution. Do not use image generation. Use source PDF crops, LaTeX/TikZ/PGFPlots, Matplotlib, Asymptote, vector drawing, or other deterministic local rendering.
- Keep the visual style minimal and suitable for an international textbook or university solutions manual, but do not use a fixed reusable layout, canned layout, fixed color scheme, or boilerplate prose.
- Perform two main-agent visual QA rounds, page by page, with rendered PDF pages inspected against the intended content and layout.
- After the two main-agent rounds, spawn exactly one independent subagent for final content QA. The subagent must perform two page-by-page content QA rounds and directly fix errors when possible.
- Do not deliver with any open content, math, LaTeX, render, or layout defect.

## Architecture

### Main Agent

The main agent owns production and the first two QA rounds:

1. Copy the source problem sheet into a job directory.
2. Extract or render source pages so every question and figure can be read.
3. Build a complete question inventory with page numbers, question numbers, subparts, required diagrams, and answer status.
4. Solve every item from first principles or from the user-provided course context. Do not skip a question because it looks repetitive or simple.
5. Create a document-specific LaTeX project for the answers.
6. Generate deterministic figures, plots, or cropped source images needed by the solutions.
7. Compile the PDF and render every answer page to a separate PNG.
8. Run visual QA pass 1 page by page. Fix defects, rebuild, and rerender affected pages.
9. Run visual QA pass 2 page by page. Fix defects, rebuild, and rerender affected pages.
10. Spawn one independent final QA subagent only after passes 1 and 2 are complete.
11. Integrate any subagent fixes, rebuild, rerender, and deliver only after final validation passes.

The main agent may use helper scripts, OCR, CAS tools, numerical checks, or symbolic algebra, but the final reasoning must remain reviewable in the LaTeX source.

When composing the LaTeX, treat appearance as a strong guidance signal: aim for a polished, minimal, high-quality university solutions-manual feel, with clear hierarchy, balanced spacing, readable derivations, well-placed figures, clean tables, and final answers that are easy to find. Let the visual treatment follow the actual problem sheet and solution content instead of reusing a fixed design.

### Final QA Subagent

The final QA subagent owns independent content verification:

1. Read the source problem sheet, the produced LaTeX project, the compiled PDF, source page renderings, and answer page renderings.
2. Reconstruct the question inventory independently enough to detect missing questions or mismatched numbering.
3. Perform content QA pass 3 page by page: verify mathematical correctness, assumptions, units, signs, boundary conditions, diagrams, references to earlier results, and completeness of each answer.
4. Directly edit the LaTeX source or deterministic figure sources for every defect it can fix.
5. Rebuild and rerender after edits.
6. Perform content QA pass 4 page by page as acceptance QA.
7. Return per-page QA records and a concise list of edits made.

The final QA subagent must not merely summarize that the PDF looks good. It must verify each page and each question/subpart against the source problem sheet.

## Artifact Contract

Each job must keep enough evidence for another agent to audit the result:

- `inputs/` contains the source problem sheet and any user-provided context.
- `source_pages/` contains one rendered source PNG per original page.
- `latex/` contains the complete LaTeX project and deterministic figure sources.
- `assets/` contains all cropped, plotted, or generated non-imagegen visual assets used by LaTeX.
- `output/<original-pdf-stem> anwser.pdf` is the compiled answer PDF. For example, a source file named `xxxx.pdf` must produce `output/xxxx anwser.pdf`. Preserve the original filename stem exactly and append the literal string ` anwser` before `.pdf`.
- `rendered/pass_1/` and `rendered/pass_2/` contain one rendered PNG per answer PDF page after each main-agent visual QA pass.
- `rendered/pass_3/` and `rendered/pass_4/` contain one rendered PNG per answer PDF page after each subagent content QA pass.
- `qa/question_inventory.json` records every source question/subpart with source page, answer location, status, and verification notes.
- `qa/pass_1_pages.json`, `qa/pass_2_pages.json`, `qa/pass_3_pages.json`, and `qa/pass_4_pages.json` contain one record per answer page checked in that pass.
- `build.log` records the final successful LaTeX build.

Every referenced image in LaTeX must exist inside the job directory. The PDF must rebuild from the delivered LaTeX project without hidden external state.

## Visual QA Rules

Every visual QA pass must inspect individual rendered pages, not contact sheets or gallery views.

Check each answer page for:

- blank pages, wrong page size, wrong orientation, or broken page order;
- missing question numbers, duplicated questions, or answers attached to the wrong problem;
- LaTeX compile artifacts, unresolved references, broken citations, `??`, `????`, replacement characters, private-use glyph boxes, or mojibake;
- formulas clipped, overflowing, or rendered as plain text when math notation is required;
- diagrams, plots, crops, or tables missing, unreadable, too small, or not referenced in the solution;
- text overlap, margin overflow, line collisions, widows that harm readability, or unreadable font size;
- visual clutter inconsistent with a minimal textbook-style answer PDF.

If any page is patched, rerender that page before recording the pass as complete.

## Content QA Rules

Check every question and subpart against the original source:

- The final PDF answers every visible prompt and preserves numbering.
- The solution states necessary assumptions and uses definitions consistent with the problem.
- Algebra, calculus, mechanics, circuits, statistics, programming, or other domain-specific steps are correct for the source question.
- Units, dimensions, significant figures, coordinate systems, sign conventions, and boundary/initial conditions are correct where applicable.
- Diagrams and plots match the variables, geometry, axes, and labels used in the derivation.
- Numerical answers are independently checked by substitution, dimensional analysis, limiting cases, or local computation when practical.
- Final answers are clearly identifiable without adding canned phrasing.

Any uncertainty about a problem statement must be resolved from the source PDF or user-provided context before delivery. Do not hide uncertainty in the final answer PDF.

## Failure Gates

- Missing source page renderings block solving.
- Missing question inventory blocks LaTeX production.
- Any unanswered question or subpart blocks delivery.
- Any LaTeX build error, warning that affects output, missing asset, or unresolved reference blocks delivery.
- Any rendered QA directory missing one PNG per answer page blocks delivery.
- Any QA JSON missing one record per checked page blocks delivery.
- Any contact sheet, montage, or combined screenshot used as primary QA evidence invalidates that QA pass.
- Any use of image generation for solution images blocks delivery.
- Any open `critical`, `major`, or `minor` defect blocks delivery.

## Delivery

Return the final answer PDF path, the LaTeX project path, and a short QA summary covering:

- final answer PDF filename follows `xxxx anwser.pdf`, where `xxxx.pdf` is the original source PDF name;
- total source pages and answer pages;
- number of questions/subparts answered;
- pass 1 and pass 2 visual QA status;
- pass 3 and pass 4 subagent content QA status;
- remaining risks, only if they are genuinely unavoidable and do not affect correctness.
