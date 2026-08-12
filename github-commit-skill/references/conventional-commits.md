# Conventional Commits 1.0.0 规范

本文件是 [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) 的完整要点整理，供生成提交信息时参照。

## 结构

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

本 skill 在 description 前插入 emoji shortcode，组合格式为：

```
<type>[(scope)]: :emoji_shortcode: <description>

[optional body]

[optional footer(s)]
```

## 核心规则

1. **type 前缀**：提交必须以 type 开头（名词，如 `feat`、`fix`），后接可选 scope、可选 `!`、必需的冒号与空格
2. **feat**：新增功能（对应 SemVer MINOR）
3. **fix**：修复 bug（对应 SemVer PATCH）
4. **BREAKING CHANGE**：在 footer 中用 `BREAKING CHANGE: <描述>`，或在 type/scope 后加 `!`（对应 SemVer MAJOR）。两者可同时使用，也可只用其一
5. **scope**：可选，括号包裹的名词，描述代码库的某个部分，如 `fix(parser):`
6. **description**：紧跟冒号空格的简短摘要
7. **body**：空一行后提供，自由格式，可多段
8. **footer**：空一行后，token + `:<space>` 或 `<space>#` + 值（git trailer 风格）
9. footer 的 token 用 `-` 替代空格（如 `Acked-by`），`BREAKING CHANGE` 例外
10. footer 的值可含空格与换行，解析至下一个合法 footer token/separator 时终止
11. 除 `BREAKING CHANGE` 外大小写不敏感；`BREAKING-CHANGE` 与 `BREAKING CHANGE` 等价

## type 列表

规范本身只强制 `feat` 和 `fix`，其余 type 来自 @commitlint/config-conventional（Angular 约定）：

| type | 说明 | SemVer |
|------|------|--------|
| `feat` | 新功能 | MINOR |
| `fix` | 修复 bug | PATCH |
| `build` | 构建系统或外部依赖变更 | — |
| `chore` | 不修改 src/test 的杂项 | — |
| `ci` | CI 配置文件与脚本 | — |
| `docs` | 仅文档变更 | — |
| `style` | 不影响代码含义的变更（空格、格式、分号等） | — |
| `refactor` | 既不修复 bug 也不新增功能的代码变更 | — |
| `perf` | 提升性能的代码变更 | — |
| `test` | 添加或修正测试 | — |
| `revert` | 回退之前的提交 | — |

## scope 规则

- **可选**：不是每个提交都需要 scope
- **名词**：描述代码库的一个部分，如 `auth`、`parser`、`api`、`ui`
- **单层**：scope 不含 `/` 或 `.`（除非项目 commitlint 配置允许）
- **推断依据**：diff 涉及的主要包/模块目录名
- **省略时机**：跨多个模块且无共性、改动太泛（如全仓格式化）时省略

## description 规则

- **现在时态**：`add` 而非 `added`
- **祈使句**：`add JWT login` 而非 `adds JWT login`
- **小写开头**：首字母小写（type 之后冒号空格后的内容）
- **不加句号**：末尾不加 `。` 或 `.`
- **聚焦变更**：描述"改了什么"而非文件名
- **具体**：含包名、版本、功能名
- **长度**：建议 ≤ 50 字符（不含 type/scope/emoji 前缀），硬上限 72 字符

## body 规则

- **何时加**：变更复杂、需要说明 WHY、或有多点变更需要分点
- **格式**：空一行后写，自由文本，可多段
- **内容**：说明"为什么改"和"怎么改的"，而非"改了什么"（description 已说）
- **行宽**：建议 ≤ 72 字符/行
- **可分点**：用 `-` 列表

**body 示例**：
```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.
```

## footer 规则

- **何时加**：有 BREAKING CHANGE、关联 issue/PR、署名审核等
- **格式**：空一行后，`token: value` 或 `token #value`
- **token 大写**：`BREAKING CHANGE`、`Reviewed-by`、`Refs`、`Closes`、`Co-authored-by`
- **token 用连字符**：多词 token 用 `-` 连接（`Acked-by`、`Co-authored-by`、`Reviewed-by`）
- **多个 footer**：每个独占一行（或段）

**footer 示例**：
```
Reviewed-by: Z
Refs: #123
Closes: #456
```

## BREAKING CHANGE 的两种写法

### 写法一：`!` 标记（推荐，视觉醒目）

```
feat(api)!: :boom: change response format to v2
```

### 写法二：footer 描述

```
feat: change response format to v2

BREAKING CHANGE: response format changed from v1 to v2, clients must update
```

### 两者同时用

```
feat(api)!: :boom: change response format to v2

BREAKING CHANGE: response format changed from v1 to v2, clients must update
```

本 skill 默认用 `!` 标记 + `:boom:` emoji。若 breaking 变更需要详细说明，额外加 footer。

## 完整示例

### 简单功能
```
feat(auth): :sparkles: add JWT-based user authentication
```

### 带 body 的修复
```
fix: :bug: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.
```

### 带 scope 与 footer
```
feat(lang): :sparkles: add Polish language support

Refs: #142
```

### 破坏性变更
```
feat(api)!: :boom: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

### 回退
```
revert: :rewind: feat(auth): add JWT-based user authentication

This reverts commit 1234567890abcdef.
```

## 与 SemVer 的关系

- `feat` → MINOR bump（如 1.2.3 → 1.3.0）
- `fix` → PATCH bump（如 1.2.3 → 1.2.4）
- `BREAKING CHANGE` / `!` → MAJOR bump（如 1.2.3 → 2.0.0）
- 其他 type（docs/chore 等）→ 不触发版本 bump

这是 CHANGELOG 自动生成与版本管理的理论基础。

## FAQ

### 多个 type 都适用时怎么办？
拆成多个提交。Conventional Commits 鼓励每个提交聚焦一个逻辑变更。若无法拆分，取主要变更的 type。

### 初始开发阶段如何处理？
按已发布产品对待。即使只有开发者使用，规范的提交信息也有助于追踪变更。

### type 大小写？
规范不强制（除 `BREAKING CHANGE` 必须大写），但建议统一小写。本 skill 统一用小写 type。

### scope 能嵌套吗？
规范未禁止，但 @commitlint/config-conventional 默认单层。若项目 commitlint 配置了 `scope-enum` 或允许嵌套，遵循项目配置。
