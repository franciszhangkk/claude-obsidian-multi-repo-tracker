# Obsidian Vault — Codex / 通用 Agent 指令

> Codex 用户：把这份文件作为 vault 根目录的 AGENTS.md（Codex 会自动加载）。
> 内容和 CLAUDE.md 基本一致，但去掉了 Claude Code 专属的 commands / hooks 引用。

## 核心原则

**先读 Obsidian，再读代码。** 不要为了理解架构去盲扫代码目录，先从 vault 拿到全貌。

## 目录结构

| 文件夹 | 用途 |
|--------|------|
| `项目/` | 每个代码仓库一个子文件夹 |
| `项目/<name>/概览.md` | 项目入口 |
| `项目/<name>/activeContext.md` | **每次开工必读** |
| `项目/<name>/progress.md` | 状态摘要 |

## 项目 ↔ 代码仓库映射

| Obsidian 路径 | 代码路径 |
|--------------|---------|
| `项目/example/` | `~/code/example/` |

## 写入约定

- 跨文件链接用全路径 `[[项目/<name>/概览]]`
- 项目文档入口统一命名 `概览.md`
- Codex 没有 commands 系统，需要手动维护文档同步

## 与 Claude Code 用户的差异

| 能力 | Claude Code | Codex |
|------|:-----------:|:-----:|
| 读 vault 结构 | ✅ | ✅ |
| `/sync-docs` 等命令 | ✅ | ❌ 需手动 |
| Stop hook 自动写日志 | ✅ | ❌ |
| AGENTS.md 自动加载 | ❌ | ✅ |
| CLAUDE.md 自动加载 | ✅ | ❌ |
