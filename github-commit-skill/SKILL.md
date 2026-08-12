---
name: github-commit-skill
description: 按规范生成 Git 提交信息并完成提交工作流。当用户想要提交代码、写 commit message、git commit、保存改动到版本库、生成提交信息、更新 CHANGELOG 时使用此 skill。即使用户只说"提交一下""commit 一下""帮我 commit""保存改动"也应触发。
---

# GitHub 提交 Skill

生成符合规范的 Git 提交信息并完成提交。提交信息采用 **Conventional Commits + emoji shortcode** 组合格式，用户可见变更同步写入 CHANGELOG.md。

## Agent 兼容性

本 skill 面向所有具备代码执行能力的 AI agent（Claude Code、WorkBuddy、Cursor、Windsurf 等），不绑定特定工具框架。

### 前置能力要求

执行本 skill 的 agent 必须能：
1. **执行 shell 命令** — 运行 `git`、`git diff`、`git commit` 等
2. **读取文件** — 检查 `AGENTS.md`、`commitlint.config.*`、`CHANGELOG.md` 等
3. **编辑/创建文件** — 更新 CHANGELOG.md

### 工具映射

下文用通用动作描述（"执行命令""读取文件""编辑文件"）。各 agent 按自身工具映射：

| 通用动作 | Claude Code | Cursor | WorkBuddy | 其他 |
|---------|-------------|--------|-----------|------|
| 执行命令 | Bash 工具 | terminal / run command | shell 执行 | agent 的命令执行能力 |
| 读取文件 | Read 工具 | 读取文件 | 文件读取 | 文件 I/O |
| 编辑文件 | Edit 工具 | 编辑文件 | 文件编辑 | 文件 I/O |
| 创建文件 | Write 工具 | 新建文件 | 文件写入 | 文件 I/O |
| 查找文件 | Glob 工具 | 文件搜索 | 文件查找 | glob/文件系统遍历 |

---

## 提交信息格式

```
<type>[(scope)]: :emoji_shortcode: <description>

[optional body]

[optional footer(s)]
```

**拆解**：
- `<type>` — Conventional Commits 类型（feat/fix/docs/refactor 等），小写
- `(scope)` — 可选，括号包裹的模块/包名（如 `auth`、`parser`）
- `:emoji_shortcode:` — 与 type 对应的 emoji shortcode（见下方映射表）
- `<description>` — 简短摘要，现在时态，小写开头，不加句号
- body — 可选，空一行后写，说明变更原因（WHY）或补充细节
- footer — 可选，空一行后写，如 `BREAKING CHANGE:`、`Refs: #123`

**Breaking change**：在 type/scope 后加 `!`，并将 emoji 替换为 `:boom:`：
```
feat(api)!: :boom: change response format to v2
```

### type → emoji 映射

| type | emoji shortcode | 说明 | CHANGELOG 区段 |
|------|----------------|------|---------------|
| `feat` | `:sparkles:` | 新功能 | Added |
| `fix` | `:bug:` | 修复 bug | Fixed |
| `refactor` | `:recycle:` | 重构，不改变功能 | Changed |
| `perf` | `:zap:` | 性能优化 | Changed |
| `docs` | `:memo:` | 文档变更 | （跳过） |
| `style` | `:art:` | 代码格式（空格、分号等） | （跳过） |
| `test` | `:white_check_mark:` | 测试相关 | （跳过） |
| `build` | `:package:` | 构建系统或依赖 | （跳过） |
| `ci` | `:construction_worker:` | CI 配置 | （跳过） |
| `chore` | `:wrench:` | 杂项，不涉及 src/test | （跳过） |
| `revert` | `:rewind:` | 回退提交 | 对应原 type 区段 |
| breaking（`!`） | `:boom:` | 破坏性变更（替换原 type emoji） | Removed |

> 完整 type 字典与选择指南见 [references/commit-types.md](references/commit-types.md)。

## 工作流

### 1. 检查 Git 状态

执行以下命令：

```bash
git status --short
git diff --staged --stat
git diff --staged
```

- 若不在 Git 仓库中 → 告知用户并停止
- 若无暂存改动 → 提示用户先 `git add`，或询问是否暂存全部改动
- `--stat` 先看改动范围概览，完整 diff 用于理解变更内容

### 2. 感知项目规范

按优先级检查项目中的提交规范文件（查找并读取）：

1. **AGENTS.md** / **CLAUDE.md**（项目根目录）— 项目自定义提交风格，**最高优先级**
2. **commitlint 配置**（`commitlint.config.*`、`.commitlintrc.*`、`package.json` 中的 `commitlint`）— type-enum / scope-enum / subject-max-length 等规则
3. **.czrc** / `package.json` 的 `config.commitizen` — 适配器规范
4. 若以上都不存在 → 使用本 skill 的默认格式（Conventional Commits + emoji shortcode）

**关键原则**：项目自定义规范优先于本 skill 默认值。若 AGENTS.md 指定了不同的 emoji 风格或格式，遵循项目规范。

### 3. 分析变更

阅读完整 diff，理解：
- **改了什么**（WHAT）— 新增功能、修复 bug、重构、文档更新等
- **为什么改**（WHY）— 从代码变更推断意图
- **影响范围**（scope）— 涉及哪些模块/包
- **是否破坏性**— 是否有 API 变更、行为变更、移除功能等

### 4. 生成提交信息

基于分析结果组装：

1. **选 type**：从映射表中选最匹配的（不确定时倾向于 `refactor` 或 `chore`，避免滥用 `feat`）
2. **选 scope**：从 diff 涉及的包/模块推断；跨多个且无共性则省略 scope
3. **选 emoji**：按 type 查映射表；breaking 用 `:boom:`
4. **写 description**：祈使句、现在时、小写开头、不加句号、聚焦变更内容而非文件名
5. **判断 body**：变更复杂或有 WHY 需要说明时加 body
6. **判断 footer**：有 BREAKING CHANGE、关联 issue 时加 footer
7. **判断 breaking**：有破坏性变更时加 `!` 并用 `:boom:`

**description 质量要求**：
- 具体而非泛泛：`add JWT-based user authentication` 优于 `update auth`
- 含具体细节（包名、版本、功能名）：`upgrade react from 18 to 19`
- 聚焦变更本身，不复述文件名
- 长度 ≤ 72 字符（含 type/scope/emoji 前缀）

### 5. 展示并确认

向用户展示生成的提交信息，格式：

```
提交信息：
feat(auth): :sparkles: add JWT-based user authentication

body（如有）：
Implement JWT token generation and validation. Tokens expire in 24h
and are stored in httpOnly cookies.

是否提交？（可修改后确认）
```

- 用户可修改 type / scope / emoji / description / body
- 用户确认后才提交
- 若用户提供了额外 context（如"这次改了 X 是为了修复 #123"），融入提交信息

### 6. 执行提交

```bash
git commit -m "$(cat <<'COMMIT_EOF'
feat(auth): :sparkles: add JWT-based user authentication

Implement JWT token generation and validation. Tokens expire in 24h
and are stored in httpOnly cookies.

Refs: #123
COMMIT_EOF
)"
```

- 多行消息用 heredoc 传递（单次 `-m` 只传单行会丢失 body/footer）
- 若 agent shell 不支持 heredoc：用多个 `-m` 参数分别传递 subject/body/footer
  ```bash
  git commit -m "feat(auth): :sparkles: add JWT-based user authentication" \
             -m "Implement JWT token generation and validation. Tokens expire in 24h and are stored in httpOnly cookies." \
             -m "Refs: #123"
  ```
- 不使用 `--no-verify`，除非用户明确要求
- 提交后执行 `git log -1 --format=full` 验证提交成功且格式正确

### 7. 更新 CHANGELOG.md

**判断是否需要更新**：仅以下类型的变更写入 CHANGELOG（用户可见变更）：
- `feat` → Added 区段
- `fix` → Fixed 区段
- breaking change（`!`）→ Removed 区段
- `perf` → Changed 区段（性能改善对用户可感知）
- `revert` → 对应原 type 的区段

其他类型（docs/style/test/build/ci/chore）默认跳过，除非用户明确要求。

**CHANGELOG 格式**：采用 [Keep a Changelog](https://keepachangelog.com/) 格式，详见 [references/changelog.md](references/changelog.md)。

**自动创建**：若项目根目录无 `CHANGELOG.md`，首次需要时自动创建（写入文件头 + `[Unreleased]` 区段）。

**更新方式**：
1. 读取现有 CHANGELOG.md
2. 在 `## [Unreleased]` 下的对应区段添加条目
3. 条目格式：`- <Description>`（CHANGELOG 不写 emoji shortcode，保持纯文本；description 内容与提交信息一致，但首字母大写）
4. 若区段不存在则新建（`### Added` / `### Fixed` 等）
5. 多条目按时间倒序（新条目在上）

**示例 CHANGELOG 条目**：
```markdown
## [Unreleased]

### Added
- Add JWT-based user authentication
- Add user profile settings page

### Fixed
- Resolve race condition in request handler
```

### 8. 验证

- `git log -1` 确认提交信息格式正确
- 若更新了 CHANGELOG，确认 CHANGELOG.md 未被加入本次提交（除非用户要求一起提交）
- 若项目有校验命令（AGENTS.md 中的 `make check-go` 等），提醒用户可运行校验，但不自动执行

## 边界情况处理

- **无暂存改动**：提示 `git add` 或询问是否暂存全部
- **大量改动**：建议用户拆分为多个提交（按功能/模块），每个提交聚焦一个逻辑变更
- **改动涉及多 type**：取主要变更的 type，或建议拆分提交
- **二进制文件 / 大文件 diff**：用 `--stat` 概览，不强行解读二进制 diff
- **首次提交**：CHANGELOG.md 自动创建，格式见 references
- **merge commit**：跳过生成，用默认 merge 消息
- **amend**：用户要求修改最近提交时，重新生成信息并 `git commit --amend`

## 参考文件

| 文件 | 何时加载 |
|------|---------|
| [references/commit-types.md](references/commit-types.md) | 不确定选哪个 type、需要 type 选择指南或完整 emoji 映射时 |
| [references/conventional-commits.md](references/conventional-commits.md) | 需要完整 Conventional Commits 规范细节、body/footer 格式规则时 |
| [references/changelog.md](references/changelog.md) | 需要完整 Keep a Changelog 格式、版本号规则、CHANGELOG 示例时 |
| [references/agent-adapters.md](references/agent-adapters.md) | 需要特定 agent 平台（Cursor、WorkBuddy 等）的适配指引时 |
