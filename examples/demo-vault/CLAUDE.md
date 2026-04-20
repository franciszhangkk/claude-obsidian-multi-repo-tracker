# Obsidian Vault — Claude Code 全局指令

这是用户的 Obsidian 知识库，作为 Claude Code 的"项目上下文缓存"。

## 核心原则

**先读 Obsidian，再读代码。** 不要为了理解架构去盲扫代码目录，先从 vault 拿到全貌。

## 目录结构

| 文件夹 | 用途 | 写入规则 |
|--------|------|---------|
| `项目/` | 每个代码仓库一个子文件夹 | 通过 `/add-project` 创建模板，手动编辑详情 |
| `项目/<name>/概览.md` | 项目入口 | 架构有重大变化时更新 |
| `项目/<name>/activeContext.md` | **每次开工必读** | 切换焦点时用 `/update-active` 更新 |
| `项目/<name>/progress.md` | 状态摘要 | 完成里程碑时更新 |
| `日记/claude-log/` | 全局工作日志（在 ~ 目录启动时写） | Claude 自动追加 |

## 项目 ↔ 代码仓库映射

| Obsidian 路径 | 代码路径 | 说明 |
|--------------|---------|------|
| `项目/example-api-server/` | `~/code/example-api-server/` | Go 后端 |
| `项目/example-web-app/` | `~/code/example-web-app/` | 前端 |
| `项目/example-shared-proto/` | `~/code/example-shared-proto/` | proto 共享库，被前两者依赖 |

## 写入约定

- 跨文件链接用全路径 `[[项目/<name>/概览]]`
- 修改代码后用 `/sync-docs` 同步对应项目文档
- 项目文档入口统一命名 `概览.md`，不要用裸数字前缀
- 标记"此区域由 X 自动维护"的部分可以覆写，其余不要碰

## 跨仓库联动规则

修改 `example-shared-proto` 后，必须：

1. 在 `项目/example-api-server/activeContext.md` 加一条"proto 已变更，待联动验证"
2. 在 `项目/example-web-app/activeContext.md` 同样加一条
3. 联动完成后再划掉

## 关键入口

- [[首页]] — vault 总入口
- [[开发工作流指南]] — 协作机制详细说明
