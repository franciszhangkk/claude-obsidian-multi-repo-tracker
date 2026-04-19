# Contributing / 贡献指南

> 中英双语 / Bilingual

---

## English

Thanks for your interest! This project is intentionally narrow in scope: **multi-repo project tracking inside an Obsidian vault, driven by Claude Code**. PRs and issues are welcome as long as they fit that scope.

### Out of scope

To keep the project focused, the following are **not** accepted (please use other tools):

- General second brain / Zettelkasten / PARA features
- Vector search / RAG / knowledge graph features
- Team collaboration features (single-user vault is the design center)
- Language-specific assumptions (Go-only, Python-only, etc.)

### How to contribute

1. **Open an issue first** for non-trivial changes (new commands, structural changes). For typos / docs, just send a PR.
2. **Templates**: if you change a template under `templates/`, update the corresponding `INSTALL.md` / `README.md` if user-visible.
3. **Commands**: each new command needs (a) a markdown file in `commands/`, (b) a row in `SKILL.md`, (c) a row in the README usage table.
4. **Codex compatibility**: don't break it (vault structure + AGENTS.md should keep working). Commands / hooks are Claude Code only — that's fine.
5. **No language lock-in**: never assume a specific tech stack in templates or commands.
6. **Commit messages**: in English or Chinese, both fine. Conventional Commits format encouraged (`feat:`, `fix:`, `docs:`, `refactor:`).

### Local development

```bash
# Clone for development
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git
cd claude-obsidian-multi-repo-tracker

# Test by symlinking to your skills dir
ln -s "$(pwd)" ~/.claude/skills/claude-obsidian-multi-repo-tracker

# Restart Claude Code, test commands
```

---

## 中文

感谢关注！本项目刻意聚焦于一件事：**用 Claude Code 在 Obsidian vault 里做跨仓库项目跟踪**。PR 和 issue 都欢迎，符合这个范围即可。

### 不接受的范围

为了保持项目聚焦，以下功能**不会**合并（请用其他工具）：

- 通用 second brain / Zettelkasten / PARA 功能
- 向量搜索 / RAG / 知识图谱
- 团队协作功能（单人 vault 是设计核心）
- 语言绑定（只支持 Go、只支持 Python 等）

### 怎么贡献

1. **重大改动先开 issue**（新命令、结构变更）。typo / 文档直接 PR。
2. **模板改了**：如果改了 `templates/` 下的模板，相应更新 `INSTALL.md` / `README.md`。
3. **新命令**需要：(a) `commands/` 下一个 markdown 文件，(b) `SKILL.md` 加一行，(c) README 用法表加一行。
4. **Codex 兼容**：别破坏（vault 结构 + AGENTS.md 要保持可用）。commands / hooks 只服务 Claude Code 是 OK 的。
5. **不绑语言**：模板和命令不要假设特定技术栈。
6. **commit 信息**：中文英文都行，推荐 Conventional Commits（`feat:`、`fix:`、`docs:`、`refactor:`）。

### 本地开发

```bash
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git
cd claude-obsidian-multi-repo-tracker

# 软链到 skills 目录测试
ln -s "$(pwd)" ~/.claude/skills/claude-obsidian-multi-repo-tracker

# 重启 Claude Code 测命令
```
