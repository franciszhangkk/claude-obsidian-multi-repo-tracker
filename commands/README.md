# Commands

> ⚠️ **MVP 占位**：当前只列出命令清单和设计意图。具体实现待迁移自 frank 现有的 `~/.claude/skills/` 下的 `/sync-docs` `/commit` `/pull`，并新写 `/init-vault` `/add-project` `/update-active` `/weekly-digest`。

## 命令清单

| 命令 | 状态 | 用途 |
|------|:---:|------|
| `init-vault.md` | 待写 | 初始化新 vault，复制 `templates/vault/` 到用户指定路径 |
| `add-project.md` | 待写 | 在 vault 里新增项目，复制 `templates/project/` 到 `<vault>/项目/<name>/` |
| `sync-docs.md` | 待迁移 | 把最近 git 变更同步到对应项目文档 |
| `commit.md` | 待迁移 | git add + commit + 自动同步文档 |
| `pull.md` | 待迁移 | git pull + 自动同步文档 |
| `update-active.md` | 待写 | 刷新当前项目的 `activeContext.md` |
| `weekly-digest.md` | 待写 | 周回顾，把日记 claude-log 编进项目 `progress.md` |

## 设计原则

1. **vault 路径不硬编码**：从 `OBSIDIAN_VAULT_PATH` 环境变量读，找不到时报错并给出配置说明
2. **不依赖飞书等外部 skill**：保持自包含
3. **语言无关**：不假设技术栈，git diff 解析也用通用方式
4. **可追溯**：每个写入操作在文档里留 `<!-- 由 /xxx 自动维护 -->` 标记

## 开发优先级

1. `init-vault` + `add-project`（用户能跑起来的最小集）
2. `sync-docs`（核心价值：代码 ↔ 文档同步）
3. `commit` + `pull`（git 操作的便捷封装）
4. `update-active` + `weekly-digest`（动态状态维护）
