---
description: 全量对账 Code ↔ Obsidian ↔ auto-memory 三方，找漂移并让用户逐项决定如何修。地图 vs 地形原则的工程兜底。
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion]
---

# /update-memory — 漂移对账 + 文档地图刷新

`/sync-docs` 是流水账（只看最新 commit，无判断），`/update-memory` 是年度审计（全量扫描，找漂移）。

**何时用**：每周一次 / 大功能完成后 / 接手老项目第一天 / 发现 Claude 行为奇怪时。

**不要用 / 不能替代的**：日常 commit 后的同步（用 `/commit` 即可）。

## 执行流程

### 1. 解析参数和定位项目

```bash
# 当前 git 仓库根
CODE_ROOT=$(git rev-parse --show-toplevel)

# 从 vault CLAUDE.md 找映射
VAULT_PATH="${OBSIDIAN_VAULT_PATH:-}"
[ -z "$VAULT_PATH" ] && echo "未配置 OBSIDIAN_VAULT_PATH，退出" && exit 1
```

支持参数：

- `/update-memory` → 当前仓库
- `/update-memory <项目名>` → 指定项目
- `/update-memory --all` → 扫所有项目（耗时较长，会提示）

### 2. 收集对账输入

并发读取：

- **Obsidian 项目目录所有 .md**：`find $VAULT_PATH/项目/<name>/ -name "*.md" -type f`
- **概览.md frontmatter**：解析 `docs:` 字段（人工维护的真相源）和"关键模块"段
- **最近 30 天 git log**：`git log --since="30 days ago" --name-only --pretty=format:"%h|%ai|%s"`
- **auto-memory 对应文件**：`~/.claude/projects/<encoded-cwd>/memory/project_*.md`（如有）
- **代码仓库 CLAUDE.md** 的"Obsidian 文档地图"段

### 3. 执行 4 类对账检查

#### 检查 A：Obsidian 引用的代码文件是否存在

扫所有 Obsidian .md 中的代码路径引用（形如 ``` `biz/agent/...` ```、`pkg/llm/...`、行内 code 含 `/` 和 `.go`/`.py`/`.proto`）：

```bash
grep -rEho '`[a-zA-Z_/][a-zA-Z0-9_/.-]+\.(go|py|proto|ts|tsx|java)`' "$VAULT_PATH/项目/<name>/"
```

对每个引用：检查 `$CODE_ROOT/<path>` 是否存在。

不存在 → 标记 **"幽灵引用"**（Obsidian 提到的代码已删/重命名）。

#### 检查 B：概览.md 的"关键模块"是否还在

读概览.md 的"关键模块"或"主要模块"表格，对每个模块路径检查 `$CODE_ROOT/<path>` 是否存在。

不存在 → 标记 **"模块漂移"**。

#### 检查 C：30 天内代码改了但 Obsidian 没改

对每个 02-功能/ 子文档：

1. 找文档中提到的代码路径（同检查 A）
2. 看这些代码文件 30 天内是否有修改（`git log --since=30d -- <path>`）
3. 看对应 02-功能/<file>.md 的修改时间（文件 mtime 或 git log）
4. 代码改了 + 文档**至少 7 天没动** → 标记 **"可能漂移"**

#### 检查 D：auto-memory 与 Obsidian 概览不一致

对照 `~/.claude/projects/<encoded-cwd>/memory/project_*.md` 内容 vs Obsidian 概览.md 的关键事实（技术栈、Agent 列表、关键模块）：

- 字段值不同 → 标记 **"memory 漂移"**

### 4. 生成对账报告

按严重度排序输出：

```
📊 漂移对账报告 — <项目名>
   扫描时间：<YYYY-MM-DD HH:MM>
   覆盖：Obsidian 24 文档 / 代码 30 天 47 提交 / auto-memory 2 文件

🔴 幽灵引用（3 处）
   1. 项目/<name>/02-功能/Skill架构/query-ad-data-skill迁移设计.md:42
      引用 `biz/agent/ad/skills/query-ad-data-xd/` — 代码已不存在
      [a] 改 Obsidian 删除/更新引用
      [b] 跳过（这是历史记录）

🟡 模块漂移（1 处）
   2. 概览.md "关键模块" 表格列出 `pkg/embedding/` — 代码里没有
      [a] 改 Obsidian 概览
      [b] 跳过

🟡 可能漂移（2 处）
   3. biz/agent/ad/skills/query-ad-data/ 30 天内 5 commits（最新 2026-04-21）
      但 02-功能/Skill架构/query-ad-data-skill迁移设计.md 上次修改 2026-03-15
      [a] 标 TODO 让用户人工确认
      [b] 自动总结代码变更追加到文档"近期更新"段
      [c] 跳过

🟢 memory 漂移（0 处）

✨ 文档地图刷新（5 处变化）
   - 新增：02-功能/<新加的文件>.md（追加到地图，desc 留 TODO）
   - 删除：05-测试与调试/<已删除的文件>.md（从地图移除）
   - 更新 desc：3 处（按 概览.md 的 docs: frontmatter）
```

### 5. 用户逐项决策

用 AskUserQuestion 一条条让用户选 [a]/[b]/[c]/跳过/全部跳过。

**重要约束**：

- 只动 Obsidian，**绝不动代码**
- 代码相关的发现只能标 TODO（用户后续自己改）
- 不批量执行 — 每条单独确认（防误操作）
- 用户选"跳过"的项目，可选记到 `<vault>/项目/<name>/.update-memory-ignore`，下次扫描忽略

### 6. 执行选定操作

按用户选择修改 Obsidian 文档：

- 删除幽灵引用 → 用 Edit 替换该行（或加 `~~删除线~~`）
- 更新概览模块表 → 用 Edit 改对应行
- 标 TODO → 在 02-功能/<file>.md 末尾追加：
  ```markdown
  ## TODO（来自 /update-memory <date>）
  - 代码 `<path>` 30 天内改了 5 次，请人工确认本文档是否需要更新
  ```

### 7. 刷新 CLAUDE.md 的"Obsidian 文档地图"段

复用 `/sync-docs` 第 8 步的逻辑（合并 frontmatter 和实际文件 → diff → 写入）。

### 8. 输出总结

```
✅ /update-memory 完成 — <项目名>

发现：
  🔴 幽灵引用 3 / 处理 2 / 跳过 1
  🟡 模块漂移 1 / 处理 1
  🟡 可能漂移 2 / 标 TODO 2
  🟢 memory 漂移 0
  ✨ 文档地图刷新 5 处变化

修改的 Obsidian 文档：
  - 项目/<name>/概览.md
  - 项目/<name>/02-功能/Skill架构/query-ad-data-skill迁移设计.md
  - <代码仓库>/CLAUDE.md（文档地图段）

下一步建议：
  - 人工 review 上述修改是否合理
  - 标 TODO 的文档需要你抽空更新
  - 如果发现误报，下次跑 /update-memory 会自动忽略已标记的项
```

## 启发式判断的阈值（可调）

| 信号 | 默认阈值 | 调整理由 |
|------|---------|---------|
| 代码 vs 文档"漂移"窗口 | 文档 7 天没动 | 太严会误报小修；太松会漏 |
| 扫描时间窗口 | 30 天 | 老项目可设 90 天 |
| 触发"批量操作建议" | 单类漂移 ≥ 5 处 | 防一次过载 |

阈值通过 `<vault>/项目/<name>/.update-memory-config.json` 调整：

```json
{
  "drift_window_days": 7,
  "scan_days": 30
}
```

## 注意事项

- **只读代码，不改代码**：本命令绝不动代码仓库的 .go/.py/.proto 文件
- **逐项确认，不批量**：每条漂移让用户单独决定
- **支持 ignore**：用户跳过的项可记入 `.update-memory-ignore`，下次跳过
- **大项目分批**：Obsidian 文档 > 50 个时分目录跑
- **失败不回滚**：每次决策成功就立刻保存（防中途失败丢决策）
- **不动 ai-agents 类老 vault 的 frontmatter**：如果概览.md 没有 `docs:` 字段，提示用户跑一次升级（不强制写入）

## 与 /sync-docs 的关系

| | `/sync-docs` | `/update-memory` |
|--|-------------|------------------|
| 触发 | 自动（commit/pull 后） | 手动 |
| 范围 | 增量（最新 1 commit） | 全量（30 天 + 全部文档） |
| 改什么 | 机器维护区（activeContext / 首页 / 最近变更 / 文档地图） | 任何文档（按用户决策） |
| 决策 | 无 | 逐项问 |
| 耗时 | 秒级 | 分钟级 |

`/sync-docs` 处理"做了什么没记录"，`/update-memory` 处理"做了什么和文档说的不一样"。两者互补，不可替代。
