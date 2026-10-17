---
name: problem-sheet-pdf-answers
description: Create fully worked English LaTeX answer PDFs for one or more problem sheets, with beginner-friendly derivations, required figures, and four page-by-page QA passes per PDF.
---

# Problem Sheet PDF Answers

## Workflow

The main agent coordinates: it assigns work, checks handoffs, collects each job's results, and delivers paths. PDF production belongs to the production subagents.

Each input problem-sheet PDF is one independent job. Apply every rule in this skill separately to each job, with its own directory, dedicated subagents, artifacts, and QA records. Each subagent handles only its assigned PDF.

For multiple PDFs, run jobs in parallel within the runtime's available agent capacity, starting queued jobs as slots become available. Keep production and final QA sequential within each job; different jobs advance independently.

Run this workflow once per PDF:

1. **Assign production — main agent.** Create the PDF's job directory, copy its source and applicable user context, and assign the job to exactly one dedicated production subagent.
2. **Inventory — production subagent.** Render every source page and inventory every visible question and subpart in original order, including footnotes, diagrams, tables, and continuation text. Record source page, question/subpart IDs, required visuals, and answer status. Continue when every source page is represented and every item is accounted for.
3. **Solve and build — production subagent.** Solve every inventory item from first principles or supplied course context, following the solution standards below. Create the complete LaTeX project, deterministic figure sources, assets, compiled PDF, and rendered answer pages. Continue when every item has a solution and every required visual is present.
4. **Production QA — production subagent.** Complete passes 1 and 2 using the QA gates below. Return the project and page records after all defects found in these passes are resolved.
5. **Independent final QA — main agent and final QA subagent.** After this PDF's production handoff, the main agent assigns exactly one independent final QA subagent dedicated to this PDF. That subagent completes passes 3 and 4 using the same gates, checks content page by page, and directly fixes errors where possible. Return the checked project and page records with any unresolved defects identified.
6. **Verify and deliver — main agent.** Review the handoff records and question inventory, verify the artifact contract, and deliver when all four passes are complete and no defects remain.

## Solution Standards

- Produce an English-only PDF from real LaTeX source, using LaTeX math for all mathematical notation. Keep the layout polished, minimal, and suitable for a university solutions manual.
- Answer every inventoried question and subpart in original order.
- Write for a student with weak mathematics foundations: define symbols, state the rules used, and show substitutions and algebra line by line, including steps usually called "standard."
- **Explain new symbols within each question.** At first use, explain every symbol introduced in the solution that is absent from that question's supplied text, equations, and figures. This includes variables, constants, functions, operators, indices, and figure labels. State what each symbol means in that question; a definition in another question or a document-wide glossary does not replace the local explanation.
- Create every requested graph, plot, sketch, construction, or diagram as complete answer content. Match the required axes, labels, variables, geometry, scale cues, and annotations, with clean, readable presentation.
- Use deterministic visuals only: source PDF crops, TikZ/PGFPlots, Matplotlib, Asymptote, or vector drawing. Image generation is excluded.

## QA Gates

In each pass, inspect every rendered answer page and check every question against the source and solution standards for:

- missing, duplicated, reordered, or mismatched questions;
- unanswered subparts or hidden continuation text;
- incorrect algebra, calculus, units, signs, assumptions, definitions, boundary conditions, or numerical results;
- unexplained symbols under the per-question rule above;
- derivations that skip steps or are too compressed for a weak mathematics student;
- missing, inaccurate, unreadable, mislabeled, or careless required visuals;
- LaTeX errors, unresolved references, missing assets, mojibake, clipped formulas, overflow, overlap, blank pages, or wrong page order.

Record each checked page in the corresponding QA file. Resolve every defect, rebuild the PDF, rerender the affected output, and inspect it again before closing the defect. Open content, math, LaTeX, render, layout, or QA defects block delivery.

## Artifact Contract

Each PDF's job directory must contain:

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

For each input PDF, identify the source file and return:

- final answer PDF path;
- LaTeX project path;
- source page count and answer page count;
- number of questions/subparts answered;
- QA status for production passes 1-2 and final QA passes 3-4;
- remaining risks only if unavoidable and not correctness-affecting.

For multiple PDFs, report each job separately. The batch is complete when every input PDF has completed its workflow and has its own verified answer PDF.
