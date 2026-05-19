# Agent Runtime Adapter

When an agent runtime reads this repository as project instructions, treat `SKILL.md` as the authoritative workflow.

- Resolve bundled files relative to this repository root.
- Create per-document job workspaces outside the repository or under ignored output folders such as `jobs/`, `tmp/`, and `output/`.
- Use real LaTeX source for final answer PDFs; do not assemble screenshot-based answer bodies.
- Preserve the artifact contract in `SKILL.md`, including source page renders, question inventory, page-by-page QA records, and final build logs.
- Use exactly one independent final QA worker when the runtime supports delegated workers. If it does not, perform the same final QA pass sequentially and record the limitation.
- Do not use image generation for solution diagrams or figures.
