# claude-obsidian-multi-repo-tracker

[中文](README.md) | [English](README-en.md)

> 一个 Claude Code Skill，把你的 Obsidian vault 变成跨多个代码仓库的项目跟踪知识库。

**核心理念**：让 Claude Code 在每次开工前先读 Obsidian，而不是从零扫代码。文档 = 上下文缓存。

---

## 为什么要这个

当你同时维护多个代码仓库（前端、后端、proto 共享库、脚手架……），Claude Code 每次进入项目都要重新理解架构、找入口、记跨仓库依赖。**结果**：每个会话浪费几千 token 在重复探索上，而且回答不一致。

这个 skill 解决的事：

- ✅ **每个项目有一份"活文档"**（架构、当前焦点、进度），Claude 开工前读 5KB 就能上手
- ✅ **跨仓库依赖一图看清**（vault 首页统一索引）
- ✅ **改完代码自动同步文档**（`/sync-docs` `/commit` `/pull`）
- ✅ **全局工作日志**自动记录每天做了什么
- ✅ **借鉴 Cline Memory Bank**，但支持多项目 + 跨仓库

---

## 它和其他方案有什么不同

| 项目 | 单项目 | 跨仓库 | 代码↔文档同步 | 中文模板 | Codex 兼容 |
|------|:----:|:----:|:----:|:----:|:----:|
| **本项目** | ✅ | ✅ | ✅ | ✅ | ⚠️ 部分 |
| [Cline Memory Bank](https://docs.cline.bot/features/memory-bank) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [obsidian-second-brain](https://github.com/eugeniughelbur/obsidian-second-brain) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [claude-obsidian (Karpathy LLM Wiki)](https://github.com/AgriciDaniel/claude-obsidian) | N/A | ❌ | ❌ | ❌ | ❌ |
| [claude-code-memory-setup](https://github.com/lucasrosati/claude-code-memory-setup) | ✅ | ❌ | ⚠️ | ❌ | ❌ |

---

## 它不是什么

- ❌ **不是** second brain / Zettelkasten / PARA —— 它只管"开发项目跟踪"，不管你的人生笔记
- ❌ **不是** RAG / 向量检索 —— 纯 markdown，靠 Claude 直接读
- ❌ **不绑定语言**—— Go / Python / Rust / JS 都能用，模板里没有任何语言特定假设

---

## 快速开始（5 分钟）

### 1. 安装 skill

```bash
git clone https://github.com/<your-account>/claude-obsidian-multi-repo-tracker.git ~/.claude/skills/claude-obsidian-multi-repo-tracker
```

### 2. 初始化你的 Obsidian vault

在 Claude Code 里：

```
/init-vault
```

它会问你 vault 路径，然后生成：

```
<你的 vault>/
├── 首页.md                    # 跨项目入口
├── 开发工作流指南.md
├── CLAUDE.md                  # 给 Claude 的全局指令
├── AGENTS.md                  # 给 Codex 的全局指令（可选）
└── 项目/                      # 项目文档目录
```

### 3. 把你的代码项目加进来

```
/add-project ai-agents ~/code/ai-agents
```

会在 `项目/ai-agents/` 下生成模板（概览、activeContext、progress、五个分类目录），并在代码仓库根写一份 `CLAUDE.md` 指向这份文档。

### 4. 日常使用

| 场景 | 命令 | 频率 |
|------|------|------|
| 改完代码同步到文档 | `/sync-docs` | 每次 commit 后（被 `/commit` 包住） |
| 提交代码 + 同步文档 | `/commit` | 每次提交 |
| 拉代码 + 更新文档 | `/pull` | 每次拉取 |
| 全量对账（找代码↔文档漂移） | `/update-memory` | 每周一次 / 大功能完成后 |

---

## v0.5：CLAUDE.md 精简为路由表（最新）

CLAUDE.md 重定位为"路由表 + 硬约束"，所有项目内容（架构、功能细节）全部下沉 Obsidian，按路由按需加载。

- **7 段新骨架**：元信息 / 项目速描 / 文档地图 / 路由表 / 真相源 / Git 约定 / 硬约束
- **路由表**：把"什么情况读什么文档"改成精确映射表，Claude 没有解释空间
- **项目速描**：必须填"谁在用、解决什么业务问题"，不只是技术栈
- **"最近变更"下沉 activeContext.md**：CLAUDE.md 不再有此段，全交 activeContext 承载
- **token 目标**：ai-agents ≈ 2K（原 4K），简单项目 500-800

---

## v0.4：文档地图 + 漂移对账

针对 dogfood 暴露的 3 个问题：

1. **CLAUDE.md 加文档地图段** — Claude 不用 Read 概览/首页就知道 Obsidian 里有什么，省 ~3K token 探路成本
2. **触发规则改为可判定清单** — 替换原来模糊的"了解架构时先读"，改成"必读场景 / 可跳过场景"具体列表
3. **地图 vs 地形原则** — Obsidian 是地图（设计意图、决策），代码是地形（实际行为）。改代码前必须 Read 实际文件，不能凭 Obsidian 描述就改。
4. **`/update-memory` 新命令** — 全量对账，找"代码改了 Obsidian 没改"的漂移，逐项让用户决策

CLAUDE.md 排版按 cache 友好顺序：稳定段（文档地图、触发规则、真相源表）放上面，易变段（最近变更）放最底部。

---

## Codex 用户

可以使用 vault 结构 + `AGENTS.md`，但 commands / hooks 是 Claude Code 专属。详见 [docs/codex-compatibility.md](docs/codex-compatibility.md)。

---

## 完整文档

- [INSTALL.md](INSTALL.md) — 详细安装与配置
- [SKILL.md](SKILL.md) — Skill 触发机制和命令清单
- [docs/对比.md](docs/对比.md) — 和其他方案的详细对比
- [docs/设计原则.md](docs/设计原则.md) — 为什么这么设计
- [examples/demo-vault/](examples/demo-vault/) — 一个最小可用的示例 vault

---

## 致谢

- 思想源头：[Andrej Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 结构参考：[Cline Memory Bank](https://docs.cline.bot/features/memory-bank)
- 自动化思路：[Self-Evolving Memory with Hooks](https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks)

## License

MIT
