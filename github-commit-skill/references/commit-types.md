# Commit Types 与 Emoji 映射

本文件提供 Conventional Commits type 的完整说明、选择指南与 emoji shortcode 映射。

## type 选择指南

选择 type 时的判断顺序：

1. **是否新增用户可感知的功能？** → `feat`
2. **是否修复了 bug？** → `fix`
3. **是否有破坏性变更（API 变更、行为变更、移除功能）？** → 对应 type + `!` + `:boom:`
4. **是否是重构，不改变功能？** → `refactor`
5. **是否是性能优化，不改变功能？** → `perf`
6. **是否仅文档变更？** → `docs`
7. **是否仅代码格式（空格、分号等）？** → `style`
8. **是否仅测试相关？** → `test`
9. **是否是构建系统或依赖变更？** → `build`
10. **是否是 CI 配置变更？** → `ci`
11. **是否回退了之前的提交？** → `revert`
12. **以上都不是，且不涉及 src/test？** → `chore`

### 选择原则

- **feat vs fix**：feat 是"新增能力"，fix 是"修复既有问题"。修复 bug 误用 feat 会让版本号多 bump 一个 MINOR。
- **refactor vs perf**：两者都不改变功能。refactor 改进结构，perf 提升性能。如果重构同时提升了性能，用 `perf`。
- **refactor vs feat**：如果重构后行为变了（即使是为了更好的实现），用 `feat` 或 `fix` 而非 `refactor`。
- **chore 的边界**：chore 是"不修改 src 或 test 文件"的杂项，如更新 gitignore、调整构建脚本。修改了 src 的杂项应归入更具体的 type。
- **不确定时**：倾向于 `refactor`（代码改动但功能不变）或 `chore`（非代码改动），避免滥用 `feat`。

## 完整映射表

| type | emoji shortcode | emoji 字符 | 说明 | 示例 |
|------|----------------|------------|------|------|
| `feat` | `:sparkles:` | ✨ | 新功能 | `feat(auth): :sparkles: add JWT-based login` |
| `fix` | `:bug:` | 🐛 | 修复 bug | `fix(api): :bug: handle null response in user endpoint` |
| `refactor` | `:recycle:` | ♻️ | 重构（不改功能） | `refactor(parser): :recycle: extract token validation logic` |
| `perf` | `:zap:` | ⚡ | 性能优化 | `perf(db): :zap: add index to user email column` |
| `docs` | `:memo:` | 📝 | 文档变更 | `docs: :memo: update API README examples` |
| `style` | `:art:` | 🎨 | 代码格式 | `style: :art: fix indentation in server.ts` |
| `test` | `:white_check_mark:` | ✅ | 测试相关 | `test(auth): :white_check_mark: add tests for token expiry` |
| `build` | `:package:` | 📦 | 构建/依赖 | `build: :package: upgrade react to 19` |
| `ci` | `:construction_worker:` | 👷 | CI 配置 | `ci: :construction_worker: add lint step to GitHub Actions` |
| `chore` | `:wrench:` | 🔧 | 杂项 | `chore: :wrench: update .gitignore` |
| `revert` | `:rewind:` | ⏪ | 回退 | `revert: :rewind: feat(auth): add JWT-based login` |
| breaking | `:boom:` | 💥 | 破坏性（替换原 type emoji） | `feat(api)!: :boom: change response format to v2` |

## breaking change 的 emoji 处理

当提交包含破坏性变更（type 后加 `!`）时，emoji 替换为 `:boom:`，**不保留原 type 的 emoji**：

```
# 正常 feat
feat(auth): :sparkles: add JWT-based login

# breaking feat
feat(api)!: :boom: change response format to v2

# breaking fix
fix(core)!: :boom: remove deprecated sync API
```

理由：`:boom:` 视觉上比 `:sparkles:` 更醒目，便于在 git log 中快速识别破坏性变更。

## emoji shortcode 语法

- 使用 GitHub Flavored Markdown 的 shortcode 语法：`:shortcode:`（两端冒号）
- shortcode 全小写，单词间用下划线：`:white_check_mark:`、`:construction_worker:`
- 这是 GitHub 在 commit message、PR、issue 中原生渲染的格式，无需额外配置
- 与 gitmoji（直接使用 emoji 字符 🐛）不同，shortcode 形式在纯文本终端中也可读

## 与 gitmoji 的区别

本项目不使用 [gitmoji](https://gitmoji.dev/)（直接用 emoji 字符），而是用 GitHub shortcode。原因：
- shortcode 在纯文本终端、git log、编辑器中均可读（`:sparkles:` 比 ✨ 更易识别）
- GitHub 在 commit message 中原生渲染 shortcode 为 emoji
- 与 AGENTS.md 中的约定一致（`:sparkles: Switch to the Go default runtime path`）
