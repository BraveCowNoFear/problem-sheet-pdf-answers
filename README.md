# Problem Sheet PDF Answers

[English](./README.md) | [简体中文](./README.zh-CN.md)

`problem-sheet-pdf-answers` is an agent skill for turning problem sheets, homework PDFs, tutorial sheets, worksheets, and exam-style handouts into fully worked English answer PDFs. It is written for Codex-style skills, but the workflow is filesystem-based and can be adapted to Claude Code, OpenCode/OpenClaw, or any runtime that can read a `SKILL.md` directory and run local tools.

The skill's core idea is strict answer production: render and inspect the source pages, inventory every question and subpart, solve the sheet in order, build the answer document from real LaTeX source, render every output page, and require multiple page-by-page QA passes before delivery.

## What Is Included

- `SKILL.md` with the full production, artifact, visual QA, content QA, and delivery contract
- `agents/openai.yaml` with Codex UI metadata
- `AGENTS.md` and `CLAUDE.md` as thin adapters for other agent environments
- `requirements.txt` with common local PDF/image helper libraries

Public releases intentionally do not include private, copyrighted, or course-specific problem sheets, source renders, answer PDFs, or worked-solution artifacts. Use only documents you are allowed to process.

## Install

Clone or copy this repository into the skills directory for your agent runtime.

Codex on Windows:

```powershell
$skills = if ($env:CODEX_HOME) { Join-Path $env:CODEX_HOME "skills" } else { Join-Path $HOME ".codex\skills" }
git clone https://github.com/BraveCowNoFear/problem-sheet-pdf-answers.git (Join-Path $skills "problem-sheet-pdf-answers")
```

Generic Unix-like layout:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone https://github.com/BraveCowNoFear/problem-sheet-pdf-answers.git "${CODEX_HOME:-$HOME/.codex}/skills/problem-sheet-pdf-answers"
```

For Claude Code, OpenCode, OpenClaw, or other filesystem-based agent environments, place the repository wherever that runtime loads skills or custom instructions. `AGENTS.md` and `CLAUDE.md` point back to `SKILL.md`.

## Requirements

- Python 3.10+
- Optional PDF/image helpers:

```bash
python -m pip install -r requirements.txt
```

- A LaTeX distribution such as TeX Live, MacTeX, or MiKTeX
- A PDF renderer such as Poppler or MuPDF for page PNG rendering
- Optional local tools for OCR, symbolic algebra, plotting, or numerical checks when the source material requires them

The skill is a workflow contract rather than a hosted service. It does not require third-party network APIs.

## Runtime Model

The skill uses neutral roles so it can adapt to different agents:

- `main agent`: coordinates only. It prepares the job workspace, copies inputs, assigns work, checks handoff records and required artifacts, integrates results, and delivers the final paths after all four QA passes are complete. It does not generate the answer PDF itself.
- `production subagent`: renders every source page, inventories questions and subparts, solves the sheet, builds the LaTeX project and deterministic figure assets, compiles and renders the answer PDF, and completes page-by-page QA passes 1–2, fixing, rebuilding, and rerendering defects before returning the completed project.
- `final QA subagent`: independently verifies question coverage and solution correctness in page-by-page content QA passes 3–4, fixes defects where possible, rebuilds, rerenders, and records the checks.

If your runtime supports subagents, follow the per-PDF workflow in `SKILL.md`: each PDF gets its own job directory, exactly one dedicated production subagent, and then exactly one dedicated independent final QA subagent. For multiple PDFs, run these jobs in parallel within available agent capacity; each subagent handles only its assigned PDF. If the runtime does not support subagents, perform the same acceptance checks sequentially for each PDF and keep the artifact record explicit.

## Artifact Contract

Each PDF's job should keep enough evidence for another agent or reviewer to audit the result:

- source problem sheet and page renders
- complete question inventory
- LaTeX source and deterministic figure assets
- compiled answer PDF
- rendered PNGs for every answer page after each QA pass
- JSON QA records for visual and content checks
- final successful LaTeX build log

See `SKILL.md` for the exact directory names and failure gates.

## Suggested GitHub Topics

`codex-skill`, `agent-skill`, `problem-sheets`, `homework`, `latex`, `pdf`, `solutions`, `visual-qa`

## License

MIT. See `LICENSE`.
