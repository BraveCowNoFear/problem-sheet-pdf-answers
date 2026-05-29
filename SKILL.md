---
name: problem-sheet-pdf-answers
description: Generate fully worked English LaTeX answer PDFs for problem sheets, with detailed beginner-friendly reasoning, required diagrams/plots, and strict page-by-page QA.
---

# Problem Sheet PDF Answers

## Requirements

- Output a final English-only PDF built from real LaTeX source.
- Answer every visible question and subpart in original order, including footnotes, diagrams, tables, and continuation text.
- Use LaTeX math for all mathematical notation.
- Write solutions for a weak mathematics student: define symbols, state rules, show substitutions and algebra line by line, and do not skip "standard" steps.
- If a prompt asks for a graph, plot, sketch, construction, or diagram, create it as required answer content. Do not replace it with prose or a minimal placeholder. Match axes, labels, variables, geometry, scale cues, and annotations, and make it clean and readable.
- Use deterministic visuals only: source PDF crops, TikZ/PGFPlots, Matplotlib, Asymptote, or vector drawing. Do not use image generation.
- Keep the PDF visually polished, minimal, and suitable for a university solutions manual.
- Do not deliver with open content, math, LaTeX, render, layout, or QA defects.

## Workflow

The main agent is the coordinator only. It assigns work, checks handoff completeness, integrates results, and delivers the final paths. It does not generate the answer PDF itself.

1. Main agent creates the job directory, copies inputs, and assigns production to exactly one production subagent.
2. Production subagent renders every source page and builds a complete question inventory with source page, question/subpart IDs, required visuals, and answer status.
3. Production subagent solves every item from first principles or supplied course context.
4. Production subagent creates the LaTeX project, deterministic figure sources, assets, compiled PDF, and rendered answer pages.
5. Production subagent performs pass 1 and pass 2 page-by-page QA, fixing, rebuilding, and rerendering defects before returning the completed project to the main agent.
6. Main agent then assigns exactly one independent final QA subagent. The final QA subagent performs pass 3 and pass 4 page-by-page content QA and directly fixes errors where possible.
7. Main agent reviews the subagent handoff records, verifies required artifacts exist, and delivers only after all four QA passes are complete.

## QA Gates

Check every page and every question for:

- missing, duplicated, reordered, or mismatched questions;
- unanswered subparts or hidden continuation text;
- incorrect algebra, calculus, units, signs, assumptions, definitions, boundary conditions, or numerical results;
- derivations that skip steps or are too compressed for a weak mathematics student;
- missing, inaccurate, unreadable, mislabeled, or careless required visuals;
- LaTeX errors, unresolved references, missing assets, mojibake, clipped formulas, overflow, overlap, blank pages, or wrong page order.

Any defect blocks delivery until fixed, rebuilt, and rerendered.

## Artifact Contract

Each job must contain:

- `inputs/`: source problem sheet and user context.
- `source_pages/`: one rendered PNG per source page.
- `latex/`: complete LaTeX project and deterministic figure sources.
- `assets/`: all crops, plots, and visual assets referenced by LaTeX.
- `rendered/pass_1/` through `rendered/pass_4/`: one PNG per answer page for each QA pass.
- `qa/question_inventory.json`: complete question/subpart inventory and verification status.
- `qa/pass_1_pages.json` through `qa/pass_4_pages.json`: one record per checked answer page.
- `build.log`: final successful LaTeX build log.
- `output/<original-pdf-stem> anwser.pdf`: final PDF. Preserve the original filename stem and append the literal string ` anwser`.

The PDF must rebuild from the delivered LaTeX project without hidden external state.

## Delivery

Return:

- final answer PDF path;
- LaTeX project path;
- source page count and answer page count;
- number of questions/subparts answered;
- QA status for production passes 1-2 and final QA passes 3-4;
- remaining risks only if unavoidable and not correctness-affecting.
