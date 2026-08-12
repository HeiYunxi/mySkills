# mySkills

> 一组面向 AI agent（Claude Code、Cursor、WorkBuddy 等）的可复用 skill 集合。
>
> A collection of reusable skills for AI agents (Claude Code, Cursor, WorkBuddy, etc.).

---

## 项目简介 / Introduction

mySkills 把日常开发中可沉淀为「技能」的工作流整理成一个个独立的 skill 文件夹,每个 skill 聚焦一个明确的任务场景(如规范化的 Git 提交、文档生成、代码审查等)。skill 以 Markdown 指令为主,不依赖外部 LLM API——agent 自身的推理与工具调用能力即可驱动。

面向人群:使用 AI agent 辅助开发的开发者,希望把团队 / 个人反复实践的工作流固化成可复用、可分享的 skill。

mySkills organizes everyday development workflows into self-contained skill folders. Each skill targets one concrete task (e.g. conventional Git commits, doc generation, code review). Skills are written as Markdown instructions and require no external LLM API — they are driven by the agent's own reasoning and tool-calling capabilities.

Target audience: developers using AI agents who want to turn proven team or personal workflows into reusable, shareable skills.

---

## Skill 清单 / Skill List

| Skill | 说明 / Description | 状态 / Status |
|-------|-------------------|---------------|
| [github-commit-skill](github-commit-skill/) | 按规范生成 Git 提交信息并完成提交工作流(Conventional Commits + emoji shortcode,同步更新 CHANGELOG) / Generate conventional Git commit messages and complete the commit workflow | ✅ 可用 / Available |

> 状态说明 / Status legend:✅ 可用 Available · 🔧 开发中 In progress · 📋 规划中 Planned

---

## 目录结构 / Directory Structure

仓库整体布局 / Repository layout:

```
mySkills/
├── README.md                # 本文件 / This file
├── LICENSE                  # MIT
├── .gitignore
├── docs/                    # 各 skill 的前期调研与设计说明 / Design docs per skill
│   └── github-commit-skill.md
├── github-commit-skill/     # 一个 skill 一个文件夹 / One folder per skill
│   ├── SKILL.md             # skill 主指令 / Main skill instructions
│   └── references/          # 按需加载的参考文档 / On-demand reference docs
│       ├── commit-types.md
│       ├── conventional-commits.md
│       ├── changelog.md
│       └── agent-adapters.md
└── <更多 skill> / <more skills>
```

单个 skill 文件夹的标准结构 / Standard structure of a single skill folder:

```
skill-name/
├── SKILL.md          # (必需 / required) YAML frontmatter + Markdown 指令 / YAML frontmatter + Markdown instructions
├── references/       # (可选 / optional) 按需加载的参考文档 / On-demand reference docs
├── scripts/          # (可选 / optional) 可执行脚本 / Executable scripts
└── assets/           # (可选 / optional) 模板、图标等 / Templates, icons, etc.
```

设计文档统一收纳在 `docs/`,以 `<skill-name>.md` 命名,供人阅读,不作为 skill 的运行部分。

Design documents live in `docs/`, named `<skill-name>.md`. They are for human reading and are not part of the skill's runtime.

---

## 使用说明 / Usage

每个 skill 独立,按需取用,无需安装整个仓库。

Each skill is standalone — pick the ones you need; you don't have to install the whole repo.

### Claude Code

将 skill 文件夹复制到 Claude 的 skills 目录:

Copy the skill folder into Claude's skills directory:

```bash
# 用户级(所有项目可用)/ User-level (available to all projects)
cp -r github-commit-skill ~/.claude/skills/

# 或项目级 / Or project-level
cp -r github-commit-skill .claude/skills/
```

复制后,Claude Code 会在匹配场景时自动触发该 skill。

After copying, Claude Code will auto-trigger the skill when the scenario matches.

### 其他 agent / Other agents

本仓库的 skill 以通用动作描述(执行命令 / 读取文件 / 编辑文件)编写,不绑定特定工具框架。各 agent 按自身工具映射即可使用,具体适配见各 skill 内的 `references/agent-adapters.md`。

Skills in this repo are written with generic actions (run command / read file / edit file) and are not tied to a specific tool framework. Each agent maps them to its own tools — see `references/agent-adapters.md` inside each skill for details.

---

## 贡献 / Contributing

欢迎通过 issue 和 PR 贡献新的 skill 或改进现有 skill。新增 skill 请遵循「一个 skill 一个文件夹」的结构,文件夹内必须有 `SKILL.md`(含 YAML frontmatter),相关调研与设计说明放至 `docs/<skill-name>.md`。

Contributions via issues and PRs are welcome. When adding a new skill, follow the "one folder per skill" structure: the folder must contain a `SKILL.md` (with YAML frontmatter), and place the related design notes in `docs/<skill-name>.md`.

---

## License

[MIT](LICENSE)
