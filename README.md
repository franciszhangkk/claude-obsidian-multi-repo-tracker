# claude-obsidian-multi-repo-tracker

[中文](README.md) | [English](README-en.md)

> 一个 Claude Code Skill：把 CLAUDE.md 变成精准的路由表，把 Obsidian 变成按需读取的项目知识库，让 Claude 每次只加载刚好够的上下文。

---

## 解决什么问题

同时维护多个代码仓库时，Claude Code 每次进入项目都要重新探索架构、找入口、记跨仓库依赖——**每个会话浪费几千 token，而且回答不一致**。

常见的解法是把所有项目知识塞进 CLAUDE.md，但这又引出新问题：**CLAUDE.md 会膨胀到 4K-10K token，每次会话全量加载，大部分内容和当前任务无关，白白占用注意力。**

这个 skill 的答案是两层分离：

| 层 | 文件 | 加载方式 | Token 预算 |
|----|------|---------|-----------|
| 导航层 | `CLAUDE.md` | 每次会话自动加载 | ~2K（路由表 + 硬约束） |
| 内容层 | Obsidian 子文档 | Claude 按路由表按需 Read | ~1-3K（只读当前任务需要的）|

---

## 核心设计

### CLAUDE.md = 路由表，不是知识库

v0.5 的 CLAUDE.md 只有 7 段，每段都有明确用途：

```
1. 元信息          ← vault 路径、关联项目（静态索引）
2. 项目速描        ← 业务定位：谁在用、解决什么问题（30 秒心智模型）
3. 文档地图        ← Obsidian 子文档清单 + 一句话说明（/sync-docs 自动维护）
4. 路由表          ← 用户意图 → 必读文档（精确映射，无解释空间）
5. 真相源原则      ← Obsidian 是地图，代码是地形
6. Git 操作约定    ← /commit /pull 优先
7. 项目硬约束      ← 不每次提醒就会犯错的强制规则
```

**不在 CLAUDE.md 里的**：架构细节、API 说明、实现模式、开发命令——这些在 Obsidian，用路由表按需找。

### Obsidian = 分层内容库

```
项目/<name>/
├── 概览.md         ← 业务定位、技术栈、关键模块（首次接触读这里）
├── activeContext.md ← 当前焦点 + 最近 5 次提交（/sync-docs 机器维护）
├── progress.md      ← 已完成功能 + 决策记录
├── 01-架构/         ← 架构设计、跨项目交互图
├── 02-功能/         ← 功能模块文档
├── 03-产品/         ← 产品需求
├── 04-参考资料/     ← API 文档、调试命令
└── 05-测试与调试/   ← Bug 记录、测试用例
```

### 地图 vs 地形原则

- **Obsidian 是地图**：设计意图、历史决策、当前焦点——代码里推不出来的东西
- **代码是地形**：函数签名、实际行为、调用链——Obsidian 可能过期，改代码前必须 Read 实际文件
- **发现不一致时**：小偏差立刻修 Obsidian / 大偏差停下来问用户 / 不要假设哪边对

---

## 和其他方案有什么不同

| 方案 | 单项目 | 跨仓库 | token 效率 | 代码↔文档同步 | 漂移对账 | 中文模板 |
|------|:----:|:----:|:----:|:----:|:----:|:----:|
| **本项目** | ✅ | ✅ | ✅ 路由按需读 | ✅ | ✅ | ✅ |
| [Cline Memory Bank](https://docs.cline.bot/features/memory-bank) | ✅ | ❌ | ❌ 每次全读 6 文件 | ❌ | ❌ | ❌ |
| [obsidian-mind](https://github.com/breferrari/obsidian-mind) | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| [claude-code-memory-setup](https://github.com/lucasrosati/claude-code-memory-setup) | ✅ | ❌ | ❌ | ⚠️ | ❌ | ❌ |

---

## 它不是什么

- ❌ **不是** second brain / Zettelkasten / PARA——只管开发项目跟踪，不管你的人生笔记
- ❌ **不是** RAG / 向量检索——纯 markdown，靠 Claude 直接读
- ❌ **不绑定语言**——Go / Python / Rust / JS 都能用，模板里没有语言特定假设
- ❌ **不依赖 hooks**——设计上主动调用命令，不做 PreToolUse / Stop hook 自动化

---

## 快速开始（5 分钟）

### 1. 安装 skill

```bash
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git \
  ~/.claude/skills/claude-obsidian-multi-repo-tracker
```

### 2. 初始化你的 Obsidian vault

```
/init-vault
```

会问你 vault 路径，生成基础结构（首页、CLAUDE.md、开发工作流指南）。

### 3. 添加你的项目

```
/add-project ai-agents ~/code/ai-agents
```

自动完成：扫描代码结构 → 生成架构草稿 → 问业务定位 → 创建 Obsidian 三件套（概览、activeContext、progress）→ 写入代码仓库 CLAUDE.md。概览的"架构速览"和"关键模块"已自动填充，不再是空白占位符。

### 4. 日常工作

| 场景 | 命令 |
|------|------|
| 提交代码 + 同步文档 + 更新焦点 | `/commit` |
| 拉代码 + 更新文档 | `/pull` |
| 只同步文档（不提交）| `/sync-docs` |
| 切换任务 / 更新当前焦点 | `/update-active` |
| 全量对账，找漂移 | `/update-memory` |

---

## 完整命令

| 命令 | 触发时机 | 说明 |
|------|---------|------|
| `/init-vault` | 通常自动调用 | 初始化 vault 结构（`/add-project` 检测到 vault 不存在时自动 inline 运行）|
| `/add-project` | 新增项目 | 检测语言 + **扫代码结构生成架构草稿** + 问业务定位 + 写 CLAUDE.md |
| `/commit` | 每次提交 | 智能暂存 + 中文 message + sync-docs + **自动推断焦点草稿** |
| `/pull` | 每次拉取 | stash 保护 + ff-only pull + 批量同步文档 |
| `/sync-docs` | 提交后 / 主动同步 | 更新 activeContext 最近变更 + 刷新文档地图 |
| `/update-active` | 切换任务时 | 从 git 信号推断焦点草稿，用户一键确认写入 activeContext |
| `/update-memory` | 每周 / 大功能后 | 全量对账 Code ↔ Obsidian，找漂移逐项决策 |

---

## Codex 用户

可以使用 vault 结构 + `AGENTS.md`，但 `/commit` `/pull` `/sync-docs` 等命令是 Claude Code 专属。详见 [docs/codex-compatibility.md](docs/codex-compatibility.md)。

---

## 完整文档

- [INSTALL.md](INSTALL.md) — 详细安装与配置
- [SKILL.md](SKILL.md) — Skill 触发机制
- [docs/设计原则.md](docs/设计原则.md) — 为什么这么设计
- [examples/demo-vault/](examples/demo-vault/) — 最小可用示例 vault

---

## 更新记录

**v0.5**：CLAUDE.md 精简为路由表 + 7 段骨架，引入项目速描（业务定位），最近变更下沉 activeContext，token 降 ~60%

**v0.4**：加文档地图段、触发规则清单、地图 vs 地形原则、`/update-memory` 命令

**v0.3**：实现 `/sync-docs` `/commit` `/pull`，完整 demo-vault，Layer 1 Git 操作约定

---

## 致谢

- 思想源头：[Andrej Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- 结构参考：[Cline Memory Bank](https://docs.cline.bot/features/memory-bank)
- 自动化思路：[Self-Evolving Memory with Hooks](https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks)

## License

MIT
