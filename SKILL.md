---
name: claude-obsidian-multi-repo-tracker
description: 把 Obsidian vault 变成跨多代码仓库的项目跟踪知识库。当用户提到 vault、知识库、项目跟踪、跨仓库依赖、文档同步时使用。
---

# claude-obsidian-multi-repo-tracker

这个 skill 让 Claude Code 在每次开工前先读 Obsidian vault 获取项目上下文，而不是从零扫代码。

## 何时触发

- 用户提到 "vault" / "Obsidian" / "知识库" / "项目文档"
- 用户在多个代码仓库间切换工作
- 用户问 "这个项目的架构" / "这个项目我做到哪了"
- 用户提交代码 / 拉取代码 / 同步文档时

## 命令清单

| 命令 | 用途 |
|------|------|
| `/init-vault` | 初始化一个新的 vault（生成首页、CLAUDE.md、AGENTS.md 等） |
| `/add-project` | 在 vault 里新增一个项目（生成模板目录） |
| `/sync-docs` | 把最近代码变更同步到 Obsidian |
| `/commit` | git commit + 自动同步文档 |
| `/pull` | git pull + 自动同步文档 |
| `/update-active` | 从 git 信号推断焦点草稿，用户一键确认后写入 activeContext.md |
| `/weekly-digest` | 周回顾，把日记编进项目 progress.md |

详见 [commands/](commands/) 目录下每个命令的说明。

## 配置

vault 路径通过环境变量配置（**不要硬编码**）：

```bash
export OBSIDIAN_VAULT_PATH="$HOME/Desktop/Obsidian Vault"
```

或在 `~/.claude/settings.json` 里：

```json
{
  "env": {
    "OBSIDIAN_VAULT_PATH": "/path/to/your/vault"
  }
}
```

## 工作机制

1. **导航**：CLAUDE.md (~2K token) 自动加载，内含路由表，告诉 Claude 什么情况读哪份文档
2. **按需读**：Claude 根据路由表按需 Read Obsidian 子文档，不是每次都全读
3. **写**：通过 commands 把代码变更/焦点更新写回 vault（/commit 顺带更新焦点）
4. **链**：跨仓库依赖通过 vault 首页统一索引，Claude 跟着 wikilink 跳转

## 不做什么

- 不做向量检索 / RAG（纯 markdown 直读）
- 不做团队协作（单人 vault，团队请用 Obsidian Sync 或 git）
- 不管 second brain / 个人笔记（只管开发项目跟踪）
