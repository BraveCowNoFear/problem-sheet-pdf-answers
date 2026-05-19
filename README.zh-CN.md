# Problem Sheet PDF Answers

[English](./README.md) | [简体中文](./README.zh-CN.md)

`problem-sheet-pdf-answers` 是一个 agent skill，用于把 problem sheet、作业 PDF、tutorial sheet、worksheet 和考试风格讲义生成完整英文解答 PDF。它兼容 Codex 的 skill 目录结构，但工作流本身基于文件系统，也可以给 Claude Code、OpenCode/OpenClaw 或任何能读取 `SKILL.md` 并运行本地工具的 agent 环境使用。

核心思路是严格生产解答：先渲染并检查原题页面，建立每道题和小问的清单，按原顺序完整求解，用真实 LaTeX 源码生成答案 PDF，逐页渲染输出，并在交付前完成多轮逐页 QA。

## 仓库内容

- `SKILL.md`：完整的生产、artifact、视觉 QA、内容 QA 和交付契约
- `agents/openai.yaml`：Codex UI 元数据
- `AGENTS.md` 和 `CLAUDE.md`：其他 agent 环境的薄适配入口
- `requirements.txt`：常用本地 PDF/图片辅助库

公开仓库不会附带私有、受版权保护或课程专属的题目 PDF、原题渲染图、答案 PDF 或解答产物。请只处理你有权使用的文档。

## 安装

把仓库 clone 或复制到你的 agent runtime 的 skills 目录。

Codex on Windows:

```powershell
$skills = if ($env:CODEX_HOME) { Join-Path $env:CODEX_HOME "skills" } else { Join-Path $HOME ".codex\skills" }
git clone https://github.com/BraveCowNoFear/problem-sheet-pdf-answers.git (Join-Path $skills "problem-sheet-pdf-answers")
```

通用 Unix-like 目录：

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone https://github.com/BraveCowNoFear/problem-sheet-pdf-answers.git "${CODEX_HOME:-$HOME/.codex}/skills/problem-sheet-pdf-answers"
```

Claude Code、OpenCode、OpenClaw 或其他基于文件系统的 agent 环境，可以把本仓库放到对应 runtime 加载 skill/custom instruction 的目录。`AGENTS.md` 和 `CLAUDE.md` 都指向 `SKILL.md`。

## 依赖

- Python 3.10+
- 可选 PDF/图片辅助库：

```bash
python -m pip install -r requirements.txt
```

- TeX Live、MacTeX 或 MiKTeX 等 LaTeX 发行版
- Poppler 或 MuPDF 等用于把 PDF 渲染成页面 PNG 的工具
- 根据题目需要，可使用本地 OCR、符号代数、绘图或数值校验工具

这个 skill 是工作流契约，不是托管服务；默认不需要第三方网络 API。

## Runtime 模型

这个 skill 用中性的角色名适配不同 agent：

- `main agent`：准备 job workspace、读取原题、求解、创建 LaTeX 项目、编译 PDF，并完成两轮视觉 QA。
- `final QA worker`：独立核对题目覆盖和解答正确性，能修就直接修，随后重编译、重渲染，并记录两轮内容 QA。

如果 runtime 支持 subagent，就严格按 `SKILL.md` 使用一个独立 final QA worker；如果不支持，就顺序执行同等验收检查，并在 artifact 记录中说明限制。

## Artifact 契约

每次运行都应保留足够证据，方便另一个 agent 或审阅者复核：

- 原题 PDF 和原题逐页渲染
- 完整 question inventory
- LaTeX 源码和确定性图表资产
- 编译后的答案 PDF
- 每轮 QA 后的答案逐页 PNG
- 视觉和内容检查的 JSON QA 记录
- 最终成功 LaTeX build log

具体目录名和失败 gate 见 `SKILL.md`。

## 建议 GitHub Topics

`codex-skill`, `agent-skill`, `problem-sheets`, `homework`, `latex`, `pdf`, `solutions`, `visual-qa`

## 许可证

MIT，见 `LICENSE`。
