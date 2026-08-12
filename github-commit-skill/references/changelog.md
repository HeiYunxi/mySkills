# CHANGELOG.md 格式与更新规则

本文件基于 [Keep a Changelog](https://keepachangelog.com/) 格式，定义 CHANGELOG.md 的结构与更新方式。

## 为什么用 Keep a Changelog

- 人类可读，非机器生成堆砌
- 按变更类型分组（Added/Fixed/Changed/Removed）
- 与 Conventional Commits 的 type 天然对应
- 与 SemVer 版本号对应
- 明确区分 `Unreleased` 与已发布版本

## 文件结构

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- :sparkles: add JWT-based user authentication

### Fixed
- :bug: resolve race condition in request handler

## [1.0.0] - 2024-01-15

### Added
- :sparkles: initial release
```

### 区段说明

| 区段 | 对应 type | 含义 |
|------|----------|------|
| `### Added` | `feat` | 新增功能 |
| `### Changed` | `refactor`, `perf` | 变更（重构、性能优化） |
| `### Deprecated` | — | 标记即将移除的功能 |
| `### Removed` | breaking `!`（移除功能） | 移除的功能 |
| `### Fixed` | `fix` | 修复的 bug |
| `### Security` | `fix`（安全相关） | 安全修复（重要安全漏洞单列） |

### 不写入 CHANGELOG 的 type

以下 type 的变更默认不写入 CHANGELOG（对用户不可见）：
- `docs` — 文档变更
- `style` — 代码格式
- `test` — 测试
- `build` — 构建/依赖（除非是用户可感知的依赖升级）
- `ci` — CI 配置
- `chore` — 杂项

## 更新规则

### 何时更新

每次提交 `feat`、`fix`、`perf`、`refactor`、breaking change 时，在 `## [Unreleased]` 下对应区段添加条目。

### 更新步骤

1. **读取现有 CHANGELOG.md**（用 Read 工具）
2. **定位 `## [Unreleased]`** 区段
3. **找到对应 type 的 `### Xxx` 子区段**（如 `### Added`）
   - 若不存在则新建（顺序：Added → Changed → Deprecated → Removed → Fixed → Security）
4. **在区段顶部添加条目**（新条目在上，时间倒序）
5. **条目格式**：`- :emoji_shortcode: <description>`
   - emoji shortcode 与提交信息一致
   - description 与提交信息的 description 一致（不含 type/scope 前缀）
6. **用 Edit 工具写入**

### 条目格式示例

提交信息：
```
feat(auth): :sparkles: add JWT-based user authentication
```

CHANGELOG 条目：
```markdown
### Added
- :sparkles: add JWT-based user authentication
```

### 带破坏性变更的条目

提交信息：
```
feat(api)!: :boom: change response format to v2

BREAKING CHANGE: response format changed from v1 to v2
```

CHANGELOG 条目（写入 `### Removed` 或 `### Changed`，视变更性质）：
```markdown
### Changed
- :boom: change response format to v2 (BREAKING)
```

## 首次创建

项目根目录无 `CHANGELOG.md` 且首次需要写入时，创建文件：

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- :sparkles: add JWT-based user authentication
```

创建步骤：
1. 用 Write 工具写入文件头与 `## [Unreleased]` 区段
2. 添加对应区段与条目

## 版本发布时的处理

当项目发布新版本时（非本 skill 自动完成，但需知晓）：
1. 将 `## [Unreleased]` 改为 `## [1.0.0] - YYYY-MM-DD`
2. 在文件顶部新增空的 `## [Unreleased]`
3. 版本号规则遵循 SemVer：
   - MAJOR：有 breaking change
   - MINOR：有 `feat`
   - PATCH：仅有 `fix`

## 区段顺序约定

保持以下顺序（即使某区段为空也按此排列）：
```
## [Unreleased]
### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security
```

## 与提交的关系

- CHANGELOG.md 的更新是**独立的改动**，不混入功能提交
- 建议：先提交功能代码，再单独提交 CHANGELOG.md 更新；或将 CHANGELOG 更新作为同一次提交的一部分（视项目惯例）
- 本 skill 默认：提交功能代码后，询问用户是否将 CHANGELOG.md 一并提交
- 若项目 AGENTS.md / CLAUDE.md 有不同约定，遵循项目约定

## 完整示例

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- :sparkles: add user profile settings page
- :sparkles: add JWT-based user authentication

### Changed
- :recycle: extract token validation into separate module
- :zap: add index to user email column

### Fixed
- :bug: resolve race condition in request handler

### Removed
- :boom: drop support for Node 6 (BREAKING)

## [1.0.0] - 2024-01-15

### Added
- :sparkles: initial release
```
