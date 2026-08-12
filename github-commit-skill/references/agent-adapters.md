# Agent 平台适配指引

本文件为不同 AI agent 平台提供本 skill 的适配要点。SKILL.md 正文已用通用动作描述（"执行命令""读取文件""编辑文件"），各平台按自身工具映射即可。本文件补充各平台的差异点与注意事项。

## 通用适配原则

1. **YAML frontmatter 可忽略**：SKILL.md 顶部的 `---` 块是 skill 元数据格式，用于触发识别。
2. **工具名称非关键**：文中出现的 "Bash 工具""Read 工具""Edit 工具" 是 Claude Code 的工具名。其他 agent 替换为自身的等价能力即可——关键是动作（执行命令 / 读写文件），不是工具名。
3. **能力前置检查**：执行前先确认 agent 具备 shell 执行 + 文件读写三项能力。若缺失任一，告知用户该平台无法完整执行本 skill。

---

## Cursor

Cursor 的 agent 模式（Composer / Chat with agent）具备终端执行与文件编辑能力。

### 工具映射

| skill 动作 | Cursor 方式 |
|-----------|------------|
| 执行命令 | terminal 命令执行 / `run command` |
| 读取文件 | 内置文件读取（agent 自动读取上下文文件） |
| 编辑文件 | 内置编辑 / multi-file edit |
| 查找文件 | `@file` 引用或文件搜索 |
| 创建文件 | 新建文件 |

### 注意事项

- Cursor Composer 倾向于直接修改代码。提交前务必先执行第 5 步「展示并确认」——不要跳过用户确认直接 commit。
- Cursor 的 `@file` / `@folder` 引用可快速引入 AGENTS.md、commitlint 配置到上下文，便于规范感知。
- Cursor 终端默认非交互式，heredoc 可能不被支持。若 heredoc 失败，改用多个 `-m` 参数：
  ```bash
  git commit -m "feat(auth): :sparkles: add JWT-based user authentication" \
             -m "Implement JWT token generation and validation." \
             -m "Refs: #123"
  ```
- Cursor Rules（`.cursorrules` 或 `.cursor/rules/*.md`）应视为与 AGENTS.md 同级的项目规范来源——在第 2 步「感知项目规范」时一并检查。

---

## WorkBuddy

### 工具映射

| skill 动作 | WorkBuddy 方式 |
|-----------|---------------|
| 执行命令 | shell 执行能力 |
| 读取文件 | 文件读取 |
| 编辑文件 | 文件编辑 |
| 查找文件 | 文件查找 / 文件系统遍历 |
| 创建文件 | 文件写入 |

### 注意事项

- 按平台实际提供的工具名称映射，核心是确保 shell、文件读写三项能力可用。
- 若 WorkBuddy 的 shell 执行对长命令有限制，将 `git diff --staged` 与后续 `git commit` 拆成独立步骤执行。
- 若平台不支持 heredoc，用多个 `-m` 参数传递多行提交信息。

---

## Claude Code

本 skill 的原生平台，frontmatter 元数据在此平台生效。

### 工具映射

| skill 动作 | Claude Code 工具 |
|-----------|----------------|
| 执行命令 | Bash |
| 读取文件 | Read |
| 编辑文件 | Edit |
| 创建文件 | Write |
| 查找文件 | Glob |

### 注意事项

- frontmatter 的 `description` 是触发机制，已覆盖"提交""commit""保存改动"等表达。
- heredoc 在 Bash 工具中可用，优先用于多行提交信息。
- 可用 Glob 查找 `commitlint.config.*`、`.czrc` 等配置文件。

---

## 其他 Agent 平台

通用适配步骤：

1. **确认能力**：平台是否支持 shell 命令执行 + 文件读写？三项缺一不可。
2. **映射工具**：将 SKILL.md 中的"执行命令""读取文件""编辑文件"映射到平台工具。
3. **测试 heredoc**：执行 `echo "$(cat <<'EOF'\ntest\nEOF\n)"` 验证是否支持。不支持则用多 `-m` 参数。
4. **检查规范感知**：确保平台能读取项目根目录的 AGENTS.md / CLAUDE.md / .cursorrules 等规范文件。
5. **保留用户确认**：无论哪个平台，第 5 步「展示并确认」不可跳过——提交是难以撤销的外向动作。

## heredoc 不可用时的通用回退

所有平台若不支持 heredoc，统一用多个 `-m` 参数。Git 会将每个 `-m` 作为独立段落，段落间空行分隔，等效于 body/footer 分段：

```bash
git commit \
  -m "feat(auth): :sparkles: add JWT-based user authentication" \
  -m "Implement JWT token generation and validation. Tokens expire in 24h and are stored in httpOnly cookies." \
  -m "Refs: #123"
```

- 第一个 `-m`：subject 行
- 第二个 `-m`：body
- 后续 `-m`：每个成为 body 的新段落或 footer
