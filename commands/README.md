# Commands

每个命令是一个 markdown 文件，里面是给 Claude Code 的指令。安装方法见 [INSTALL.md](../INSTALL.md)。

## 命令清单

| 命令 | 状态 | 用途 |
|------|:---:|------|
| [`init-vault.md`](init-vault.md) | ✅ MVP | 初始化新 vault，复制 `templates/vault/` 到用户指定路径 |
| [`add-project.md`](add-project.md) | ✅ MVP | 在 vault 里新增项目，自动检测语言/依赖，更新首页索引和代码仓库 CLAUDE.md |
| [`sync-docs.md`](sync-docs.md) | ✅ MVP | 把最近 git 变更同步到对应 Obsidian 项目的 `activeContext.md` 和首页 |
| [`commit.md`](commit.md) | ✅ MVP | git add + commit（智能 message）+ 自动调用 `/sync-docs` |
| [`pull.md`](pull.md) | ✅ MVP | git pull（带工作区检查）+ 批量同步新提交到 Obsidian |
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
