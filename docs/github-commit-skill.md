---
name: github-commit-skill-design-notes
description: "Design notes and research for the github-commit-skill. Read this to understand the skill's background, the surveyed AI-commit tools (aicommits, opencommit, commitizen), the Conventional Commits spec, the Claude Code skill format, and the resulting capability targets. This is a human-readable design document, not a runtime skill."
---
# 前期调研与设计说明：github-commit-skill

> 本文档是「GitHub 提交 Skill」的前期调研与设计说明。目标是为 Claude Code / Agent 形态的 skill 奠定基础，使其能够高质量地生成 Git 提交信息并完成提交工作流。
>
> 调研对象：GitHub 上现有的 AI commit 工具（aicommits、opencommit、commitizen/cz-cli）、Conventional Commits 1.0.0 规范、Claude Code skill 文件格式，以及用户提供的项目文档（`AGENTS.md`）。

---

## 一、调研背景与目标

### 1.1 任务来源
用户希望新建一个 skill 文件夹，目标是编写一个「GitHub 提交 skill」——让 Claude Code 在具备该 skill 后，能够按规范生成提交信息并完成 Git 提交。在动手写 skill 之前，先在 GitHub 上搜索现有相关项目的模式、功能、特性、可复用内容，结合用户提供的文档，总结形成本说明文档。

### 1.2 成功标准
- 摸清主流 AI commit 工具的核心功能、工作流、提示词模式、配置方式
- 梳理 Conventional Commits 规范要点
- 明确 Claude Code skill 的文件格式与组织方式
- 提炼可复用的设计要素，给出 skill 的目标能力清单与设计方向
- 形成可供后续 skill 编写直接参照的说明文档

---

## 二、GitHub 现有项目调研

### 2.1 aicommits（Nutlope/aicommits）

**定位**：CLI 工具，用 AI 为 staged changes 生成 commit message。

**核心功能**：
- 读取 `git diff --staged`，调用 LLM 生成提交信息
- 支持生成多条候选信息供选择（`--generate <i>`，默认 1 条）
- 支持三种提交格式：`plain` / `conventional` / `gitmoji`（`--type`）
- 支持自定义 prompt 引导 LLM 行为（`--prompt`，如指定语言、风格、关注点）
- 可复制到剪贴板而非直接提交（`--clipboard`）
- 可跳过确认直接提交（`--yes`）
- 可排除特定文件（`--exclude`）
- 绕过 pre-commit hook（`--no-verify`）
- 提供 git hook（`prepare-commit-msg`）集成方式，保留原生 `git commit` 体验

**工作流模式**：
```
git add <files>
aicommits            # 生成消息 → 确认 → 提交
aicommits -g 3       # 生成 3 条候选 → 选择 → 提交
aicommits hook install  # 安装 prepare-commit-msg hook
```

**配置方式**：
- `aicommits setup` 交互式配置：选 provider → 配 API key → 自动拉取可用模型 → 选提交格式
- 配置文件：`~/.aicommits`
- 支持环境变量 / `aicommits config set` 命令
- 支持的 provider：TogetherAI、OpenAI、Groq、xAI、OpenRouter、Ollama（本地）、LM Studio（本地）、任意 OpenAI 兼容端点
- Node.js ≥ 22

**提示词模式（源码 `src/utils/prompt.ts`）**：
核心提示词由多个片段拼接：
```
Generate a concise git commit message title in present tense that precisely describes
the key changes in the following code diff. Focus on what was changed, not just file names.
Provide only the title, no description or body.
Message language: ${locale}
Commit message must be a maximum of ${maxLength} characters.
Exclude anything unnecessary such as translation. Your entire response will be passed directly into git commit.
IMPORTANT: Do not include any explanations... Respond with ONLY the commit message text.
Be specific: include concrete details (package names, versions, functionality) rather than generic statements.
${customPrompt}
${commitTypes[type]}        // conventional 模式下注入 type→description 的 JSON 字典
${specifyCommitFormat(type)} // 输出格式约束
```

**conventional 模式的 type 字典**（带描述，要求 type 小写）：
`docs`、`style`、`refactor`、`perf`、`test`、`build`、`ci`、`chore`、`revert`、`feat`、`fix`

**gitmoji 模式**：内置 60+ emoji→description 映射表。

**可复用特性**：
- 多候选生成 + 选择的交互模式
- 三种格式（plain/conventional/gitmoji）切换
- 自定义 prompt 注入
- prepare-commit-msg hook 集成
- 对推理模型 `<think>` 标签的清洗逻辑
- 消息清洗：去引号、取首行、去尾句号
- 超长消息二次缩短（LLM 调用）
- 去重

---

### 2.2 OpenCommit（di-sukharev/opencommit）

**定位**：CLI 工具（命令 `oco`），Auto-generate meaningful commits with AI。GitHub 2023 hackathon 获奖项目。

**核心功能**：
- 读取 staged diff，生成提交信息（可自动 `git add`）
- 深度集成 `@commitlint`：读取项目 commitlint 配置，自动推断 type/scope 规则并生成一致性 prompt
- 支持 GitMoji（默认 10 个，`--fgm` 启用全量 60+ 规范）
- `--yes` 跳过确认直接提交
- `OCO_DESCRIPTION` 在消息后追加 3 句「为什么」描述
- `OCO_WHY` 输出变更原因（WIP，计划用 RAG 实现）
- `OCO_ONE_LINE_COMMIT` 单行提交
- `OCO_OMIT_SCOPE` 省略 scope
- 多语言（`OCO_LANGUAGE` locale）
- 模型管理：`oco setup` 自动拉取 provider 可用模型，缓存 7 天；`oco models` 查看 / `--refresh` 刷新

**工作流模式**：
```
git add <files>
oco                  # 生成 → 确认 → 提交
oco --yes             # 跳过确认
oco --fgm             # 全量 gitmoji
```

**配置方式**：
- 全局：`~/.opencommit`，`oco config set KEY=VALUE`
- 本地（per-repo）：项目 `.env` 文件，优先级高于全局
- `oco config describe [KEY]` 查看配置项说明
- 关键配置：
  - `OCO_AI_PROVIDER`：openai / anthropic / azure / ollama / llamacpp / gemini / flowise / deepseek / aimlapi
  - `OCO_API_KEY`、`OCO_API_URL`、`OCO_API_CUSTOM_HEADERS`
  - `OCO_TOKENS_MAX_INPUT`（默认 4096）、`OCO_TOKENS_MAX_OUTPUT`（默认 500）
  - `OCO_MODEL`、`OCO_EMOJI`、`OCO_LANGUAGE`、`OCO_DESCRIPTION`、`OCO_ONE_LINE_COMMIT`、`OCO_PROMPT_MODULE`（conventional-commit / @commitlint）

**提示词模式（源码 `src/prompts.ts`）**：
采用 system message 结构，由多个语义片段组装：
- **身份**：`You are to act as an author of a commit message in git.`
- **使命**：`create clean and comprehensive commit messages as per the ${convention} and explain WHAT were the changes and mainly WHY the changes were done.`
- **diff 指令**：`I'll send you an output of 'git diff --staged' command, and you are to convert it into a commit message.`
- **规范约束**：conventional 关键词列表 / gitmoji 帮助表
- **描述指令**：是否追加 WHY 描述
- **单行指令**：OCO_ONE_LINE_COMMIT
- **scope 指令**：是否省略 scope
- **通用约束**：`Use the present tense. Lines must not be longer than 74 characters. Use ${language} for the commit message.`
- **用户上下文**：支持注入额外 context（`<context>...</context>`）
- 内置一个 `INIT_DIFF_PROMPT` 示例 diff 作为 few-shot

**commitlint 集成（源码 `src/modules/commitlint/config.ts`）**：
- 读取项目 `@commitlint` 配置 → `inferPromptsFromCommitlintConfig` 推断规则 → 生成一致性 prompt → 调 LLM 生成 consistency JSON → 缓存（带 hash，配置不变则复用）
- 这是 OpenCommit 区别于 aicommits 的核心差异化能力

**可复用特性**：
- 读取并遵循项目 commitlint 配置（type-enum、scope-enum 等规则）
- system message 的模块化组装（身份 + 使命 + diff 指令 + 规范 + 通用约束 + 用户上下文）
- 「WHAT + WHY」双维度描述
- 74 字符行宽约束
- 用户额外 context 注入
- 全局 / 本地双层配置 + 优先级
- 模型自动发现与缓存

---

### 2.3 Commitizen / cz-cli（commitizen/cz-cli）

**定位**：非 AI 的交互式提交规范工具。用 `git cz` 替代 `git commit`，通过交互式 prompt 引导填写符合规范的提交信息。

**核心功能**：
- 交互式问卷：type → scope → subject → body → breaking change → footer
- 通过 adapter（适配器）机制支持不同规范，默认 `cz-conventional-changelog`（Angular 约定）
- 校验提交格式，即时反馈
- 支持 git hook（`prepare-commit-msg` + `--hook`）强制规范
- 与 husky 等 hook 管理工具兼容

**工作流模式**：
```
git cz               # 交互式问答 → 生成规范提交
npx cz               # 免全局安装
git commit           # 配合 prepare-commit-msg hook 自动触发 cz --hook
```

**配置方式**：
- 全局安装 `commitizen` + adapter
- 项目级：`commitizen init cz-conventional-changelog --save-dev --save-exact`
  - 写入 `package.json` 的 `config.commitizen.path`
  - 或 `.czrc` 文件 `{"path": "cz-conventional-changelog"}`
- adapter 解析支持：npm 模块 / 目录 / 文件名 / 绝对路径（`require.resolve`）
- 本地安装 + npm script `"commit": "cz"` 供团队成员统一版本

**可复用特性**：
- adapter 插件化机制（规范可替换）
- 交互式字段引导（type → scope → subject → body → footer）
- `prepare-commit-msg` hook 强制规范
- 项目级 `.czrc` / `package.json config` 声明规范
- 「Commitizen friendly」badge 机制

---

## 三、Conventional Commits 1.0.0 规范要点

> 来源：conventionalcommits.org 官方规范（v1.0.0）

### 3.1 结构

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 3.2 核心规则
1. 提交必须以 **type** 开头（名词，如 `feat`、`fix`），后接可选 scope、可选 `!`、必需的冒号与空格
2. `feat`：新增功能（对应 SemVer MINOR）
3. `fix`：修复 bug（对应 SemVer PATCH）
4. **BREAKING CHANGE**：在 footer 中用 `BREAKING CHANGE: <描述>`，或在 type/scope 后加 `!`（对应 SemVer MAJOR）
5. **scope**：可选，括号包裹的名词，描述代码库的某个部分，如 `fix(parser):`
6. **description**：紧跟冒号空格的简短摘要
7. **body**：空一行后提供，自由格式多段
8. **footer**：空一行后，token + `:<space>` 或 `<space>#` + 值（git trailer 风格），token 用 `-` 替代空格（如 `Acked-by`），`BREAKING CHANGE` 例外
9. 除 `BREAKING CHANGE` 外大小写不敏感；`BREAKING-CHANGE` 与 `BREAKING CHANGE` 等价

### 3.3 推荐的 type（来自 @commitlint/config-conventional / Angular 约定）
`feat`、`fix`、`build`、`chore`、`ci`、`docs`、`style`、`refactor`、`perf`、`test`、`revert`

### 3.4 示例
```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```
```
feat(api)!: send an email to the customer when a product is shipped
```
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Reviewed-by: Z
Refs: #123
```

### 3.5 价值
- 自动生成 CHANGELOG
- 自动判定语义化版本 bump
- 向团队/公众传达变更性质
- 触发构建与发布流程
- 结构化提交历史，降低贡献门槛

---

## 四、Claude Code Skill 形态调研

### 4.1 Skill 文件格式
基于本地 `skill-creator` skill 及现有 skill（docx、web-access 等）的观察：

```
skill-name/
├── SKILL.md              (必需) YAML frontmatter + Markdown 指令
├── scripts/              (可选) 可执行脚本，确定性/重复任务
├── references/           (可选) 按需加载的参考文档
└── assets/               (可选) 输出用模板、图标等
```

**YAML frontmatter（必需字段）**：
```yaml
---
name: skill-identifier
description: "Trigger conditions + capability description. This is the main trigger mechanism — combine 'what it does' and 'when to use'. All 'when to use' info goes here, not in the body."
---
```

### 4.2 渐进式披露（三级加载）
1. **元数据**（name + description）：始终在上下文中（~100 词）
2. **SKILL.md 正文**：skill 触发时加载（建议 <500 行）
3. **打包资源**：按需加载（无限制；脚本可不加载直接执行）

### 4.3 编写要点
- description 要「略带推动性」以避免欠触发，覆盖用户可能的表达方式
- 正文用祈使句
- 解释「为什么」优于堆砌 MUST
- 大型参考文件（>300 行）加目录
- 多领域时按变体组织（如 `references/aws.md`、`references/gcp.md`），只读相关文件
- 可定义输出格式模板
- 可包含示例（Input/Output 模式）

### 4.4 与 CLI 工具的关键差异

| 维度 | CLI 工具（aicommits/opencommit） | Claude Code Skill |
|------|------|------|
| 运行方式 | 独立进程，调用外部 LLM API | Claude 本身即是 LLM，无需外部调用 |
| 工具调用 | 无 | 可调用 Bash（git）、Read、Edit 等工具 |
| 上下文 | 仅 git diff | 可读取整个仓库、文件、AGENTS.md、commitlint 配置、历史提交 |
| 交互 | CLI prompt 交互 | 自然语言对话 |
| 配置 | 配置文件 + 环境变量 | skill 指令 + 项目文件（AGENTS.md 等） |
| 提示词 | 硬编码在源码 | 写在 SKILL.md，可读可改 |
| 验证 | 需自行实现 | 可用 Read 验证提交结果、读 commitlint 校验 |

**关键洞察**：Claude Code skill 的优势在于——Claude 自身具备强推理与代码理解能力，无需把 diff 塞进 API prompt；可直接用工具读取仓库任意文件、理解变更意图、验证提交结果。因此 skill 应聚焦于「工作流编排 + 规范约束 + 项目上下文感知」，而非「如何调用 LLM」。

---

## 五、用户项目文档（AGENTS.md）要点

用户提供的 `AGENTS.md`（来自 `nexus-main` 项目）定义了该项目对 Agent 的提交规范：

### 5.1 提交风格
> Use English commit messages with an emoji prefix, for example `:sparkles: Switch to the Go default runtime path`. Keep user-visible changes reflected in `CHANGELOG.md`.

即：**英文提交信息 + emoji 前缀**（`:sparkles:` 这种 shortcode 形式），且用户可见的变更需反映到 `CHANGELOG.md`。

### 5.2 对 skill 的启示
- 不同项目有不同的提交规范（本项目用 emoji shortcode，不是 conventional commits 也不是 gitmoji emoji 字符）
- skill 必须能**感知并遵循项目级提交规范**（优先读取 `AGENTS.md` / `CLAUDE.md` / `.claude` 配置 / commitlint 配置）
- 还需兼顾 `CHANGELOG.md` 的同步更新
- 项目有明确的构建/校验命令体系（`make check-go` 等），提交前可能需要跑校验

---

## 六、可复用内容提炼

综合上述调研，以下是可直接复用到 skill 设计中的要素：

### 6.1 规范知识（可放入 references/）
- **Conventional Commits 1.0.0 结构与规则**（type/scope/`!`/BREAKING CHANGE/footer）
- **type 字典**（feat/fix/docs/style/refactor/perf/test/build/ci/chore/revert 及描述）——aicommits 的 JSON 字典是很好的参考
- **gitmoji 映射表**（aicommits 60+ 条 / opencommit 全量规范）
- **行宽约束**：OpenCommit 用 74 字符；Conventional Commits 惯例 subject ≤ 50-72 字符，body ≤ 72 字符

### 6.2 工作流模式（可复用）
1. 读取 staged diff（`git diff --staged`）
2. 感知项目规范（AGENTS.md / CLAUDE.md / commitlint / .czrc）
3. 生成提交信息（遵循项目规范，默认 conventional）
4. 多候选 / 单候选 + 用户确认
5. 执行提交（`git commit -m "..."`）
6. 验证（读 commit log / 跑校验）
7. 可选：同步 CHANGELOG.md

### 6.3 提示词设计原则（从 aicommits + opencommit 提炼）
- 祈使句、现在时
- 聚焦「改了什么」而非文件名
- 具体而非泛泛（含包名、版本、功能）
- 仅输出提交信息本身，不加解释
- 长度约束
- WHAT + WHY 双维度（opencommit）
- 注入项目规范约束（type-enum、scope 规则等）
- 支持用户额外 context

### 6.4 配置感知（从 opencommit + commitizen 提炼）
- commitlint 配置（`@commitlint/load`）→ type-enum / scope-enum / max-length
- `.czrc` / `package.json config.commitizen` → adapter 规范
- `AGENTS.md` / `CLAUDE.md` → 项目自定义提交风格（如 emoji shortcode）
- `.gitmessage` 模板
- 全局 git config（`commit.template`）

### 6.5 清洗与验证逻辑（从 aicommits 提炼）
- 去除 `<think>` 标签（推理模型）
- 去引号、取首行、去尾句号
- 去重
- 超长消息二次缩短
- 提交后用 `git log -1` 验证

---

## 七、Skill 目标能力清单（设计方向）

基于调研，初步构想该 skill 应具备以下能力（供后续 SKILL.md 编写参照）：

### 7.1 核心能力
1. **Diff 获取**：通过 Bash 执行 `git diff --staged`（或 `git status` 判断是否有暂存）
2. **规范感知**：按优先级读取项目提交规范——`AGENTS.md` / `CLAUDE.md` → commitlint 配置 → `.czrc` → 默认 Conventional Commits
3. **信息生成**：基于 diff + 项目规范生成提交信息（默认 conventional；项目指定 emoji shortcode 则遵循之）
4. **多格式支持**：conventional / plain / gitmoji / 项目自定义（如 emoji shortcode）
5. **用户确认**：生成后展示候选，用户确认或修改后再提交
6. **提交执行**：`git commit` 提交
7. **结果验证**：读 `git log -1` 确认提交成功
8. **CHANGELOG 同步**（按需）：若项目要求（如 AGENTS.md 中提到），同步更新 CHANGELOG.md

### 7.2 增强能力
- 多候选生成供选择
- 支持「WHY」描述（body 说明变更原因）
- 支持用户额外 context 注入（「这次改了 X 是为了 Y」）
- 提交前校验提醒（按 AGENTS.md 的构建/校验命令）
- scope 智能推断（从 diff 涉及的包/模块）
- breaking change 检测与标注
- 中文 / 多语言提交信息（按项目要求）

### 7.3 设计原则
- **项目规范优先**：始终先读项目文件，遵循项目既有规范，不强行套用 conventional
- **可解释**：生成信息时可说明依据（基于哪个规范、为何选这个 type）
- **最小侵入**：不修改用户未暂存的改动，提交前确认
- **渐进披露**：SKILL.md 放工作流与核心规则，规范细节放 `references/`
- **不强加外部依赖**：利用 Claude 自身能力 + 内置工具，不要求用户配置 API key

---

## 八、建议的 skill 目录结构

```
github-commit-skill/
├── SKILL.md                     # 主指令：工作流 + 规范感知 + 生成原则
└── references/
    ├── conventional-commits.md  # Conventional Commits 1.0.0 规范详解
    ├── commit-types.md          # type 字典（含描述）
    ├── gitmoji.md              # gitmoji 映射表
    └── project-conventions.md  # 如何识别与遵循项目级提交规范
                                 # （AGENTS.md / commitlint / .czrc / .gitmessage）
```

---

## 九、参考来源

- [Nutlope/aicommits](https://github.com/Nutlope/aicommits) — AI commit CLI，源码 `src/utils/prompt.ts`、`src/utils/openai.ts`
- [di-sukharev/opencommit](https://github.com/di-sukharev/opencommit) — OpenCommit CLI，源码 `src/prompts.ts`、`src/modules/commitlint/config.ts`
- [commitizen/cz-cli](https://github.com/commitizen/cz-cli) — Commitizen 交互式提交规范工具
- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) — 提交信息规范
- 本地 `skill-creator` skill（`~/.claude/skills/skill-creator/SKILL.md`）— Claude Code skill 文件格式与编写指南
- 用户提供的 `AGENTS.md`（`D:\MyLaptop\Downloads\nexus-main\nexus-main\AGENTS.md`）— 项目提交规范示例

---

## 十、下一步

本说明文档完成后，可进入 skill 编写阶段：
1. 依据第七节能力清单与第二节设计原则编写 `SKILL.md`
2. 将规范细节（第三节）拆分到 `references/` 下
3. 用 `skill-creator` 的流程跑测试用例验证（2-3 个真实场景）
4. 迭代优化
