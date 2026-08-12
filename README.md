<div align="center">

[**English**](#english) &nbsp;|&nbsp; [**中文**](#chinese)

</div>

---

<h1 id="english" align="center">mySkills</h1>

<p align="center">A collection of reusable skills for AI agents (Claude Code, Cursor, WorkBuddy, etc.)</p>

## Introduction

mySkills organizes everyday development workflows into self-contained skill folders. Each skill targets one concrete task (e.g. conventional Git commits, doc generation, code review). Skills are written as Markdown instructions and require no external LLM API — they are driven by the agent's own reasoning and tool-calling capabilities.

Target audience: developers using AI agents who want to turn proven team or personal workflows into reusable, shareable skills.

## Skill List

| Skill | Description | Status |
|-------|-------------|--------|
| [github-commit-skill](github-commit-skill/) | Generate conventional Git commit messages and complete the commit workflow | ✅ Available |

> Status legend: ✅ Available · 🔧 In progress · 📋 Planned

## Directory Structure

Repository layout:

```
mySkills/
├── README.md
├── LICENSE                  # MIT
├── .gitignore
├── docs/                    # Design docs per skill
│   └── github-commit-skill.md
├── github-commit-skill/     # One folder per skill
│   ├── SKILL.md             # Main skill instructions
│   └── references/          # On-demand reference docs
│       ├── commit-types.md
│       ├── conventional-commits.md
│       ├── changelog.md
│       └── agent-adapters.md
└── <more skills>
```

Standard structure of a single skill folder:

```
skill-name/
├── SKILL.md          # (required) YAML frontmatter + Markdown instructions
├── references/       # (optional) On-demand reference docs
├── scripts/          # (optional) Executable scripts
└── assets/           # (optional) Templates, icons, etc.
```

Design documents live in `docs/`, named `<skill-name>.md`. They are for human reading and are not part of the skill's runtime.

## Usage

Each skill is standalone — pick the ones you need; you don't have to install the whole repo.

### Claude Code

Copy the skill folder into Claude's skills directory:

```bash
# User-level (available to all projects)
cp -r github-commit-skill ~/.claude/skills/

# Or project-level
cp -r github-commit-skill .claude/skills/
```

After copying, Claude Code will auto-trigger the skill when the scenario matches.

### Other agents

Skills in this repo are written with generic actions (run command / read file / edit file) and are not tied to a specific tool framework. Each agent maps them to its own tools — see `references/agent-adapters.md` inside each skill for details.

## License

[MIT](LICENSE)

---

<h1 id="chinese" align="center">mySkills</h1>

<p align="center">一组面向 AI agent（Claude Code、Cursor、WorkBuddy 等）的可复用 skill 集合</p>

## 项目简介

mySkills 把日常开发中可沉淀为「技能」的工作流整理成一个个独立的 skill 文件夹，每个 skill 聚焦一个明确的任务场景（如规范化的 Git 提交、文档生成、代码审查等）。skill 以 Markdown 指令为主，不依赖外部 LLM API——agent 自身的推理与工具调用能力即可驱动。

面向人群：使用 AI agent 辅助开发的开发者，希望把团队 / 个人反复实践的工作流固化成可复用、可分享的 skill。

## Skill 清单

| Skill | 说明 | 状态 |
|-------|------|------|
| [github-commit-skill](github-commit-skill/) | 按规范生成 Git 提交信息并完成提交工作流（Conventional Commits + emoji shortcode，同步更新 CHANGELOG） | ✅ 可用 |

> 状态说明：✅ 可用 · 🔧 开发中 · 📋 规划中

## 目录结构

仓库整体布局：

```
mySkills/
├── README.md
├── LICENSE                  # MIT
├── .gitignore
├── docs/                    # 各 skill 的前期调研与设计说明
│   └── github-commit-skill.md
├── github-commit-skill/     # 一个 skill 一个文件夹
│   ├── SKILL.md             # skill 主指令
│   └── references/          # 按需加载的参考文档
│       ├── commit-types.md
│       ├── conventional-commits.md
│       ├── changelog.md
│       └── agent-adapters.md
└── <更多 skill>
```

单个 skill 文件夹的标准结构：

```
skill-name/
├── SKILL.md          # （必需）YAML frontmatter + Markdown 指令
├── references/       # （可选）按需加载的参考文档
├── scripts/          # （可选）可执行脚本
└── assets/           # （可选）模板、图标等
```

设计文档统一收纳在 `docs/`，以 `<skill-name>.md` 命名，供人阅读，不作为 skill 的运行部分。

## 使用说明

每个 skill 独立，按需取用，无需安装整个仓库。

### Claude Code

将 skill 文件夹复制到 Claude 的 skills 目录：

```bash
# 用户级（所有项目可用）
cp -r github-commit-skill ~/.claude/skills/

# 或项目级
cp -r github-commit-skill .claude/skills/
```

复制后，Claude Code 会在匹配场景时自动触发该 skill。

### 其他 agent

本仓库的 skill 以通用动作描述（执行命令 / 读取文件 / 编辑文件）编写，不绑定特定工具框架。各 agent 按自身工具映射即可使用，具体适配见各 skill 内的 `references/agent-adapters.md`。

## License

[MIT](LICENSE)
