---
description: 把当前代码仓库的最近 git 提交同步到 Obsidian 对应项目文档。提交后使用，或被 /commit /pull 自动调用。
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion]
---

# /sync-docs — 项目文档同步

把当前 git 仓库的最近变更同步到对应 Obsidian 项目的 `activeContext.md`，并刷新 `首页.md` 的项目活跃状态。

## 执行流程

### 1. 解析 vault 路径

按优先级取：

1. 命令参数 `$ARGUMENTS`（如 `/sync-docs ~/Documents/MyVault`）
2. 环境变量 `$OBSIDIAN_VAULT_PATH`
3. 都没有 → 报错并提示用户配置：

   ```
   未找到 Obsidian vault。请：
   - 在 ~/.claude/settings.json 的 env 里设置 OBSIDIAN_VAULT_PATH，或
   - 直接传参：/sync-docs <vault 路径>
   ```

   **不要随便猜默认路径**。

### 2. 匹配当前项目

```bash
git rev-parse --show-toplevel    # CODE_ROOT
```

读取 `$VAULT_PATH/CLAUDE.md`，解析"项目 ↔ 代码仓库映射"表（markdown 表格），找到 `代码路径` 列等于 `CODE_ROOT`（或 `CODE_ROOT` 是其前缀展开）的那一行，取出 `Obsidian 路径`（形如 `项目/<name>/`）。

匹配不到就用 AskUserQuestion 让用户从已存在的项目列表里选，或者提示用 `/add-project` 先注册。

### 3. 收集最近一次提交

```bash
git log -1 --format="%h|%s|%ai|%an"
git diff HEAD~1 --name-only
```

如果是首次提交（没有 HEAD~1），用 `git show --name-only HEAD` 取文件列表。

### 4. 检测架构变更

把变更文件名跟以下模式做匹配（语言无关，覆盖大多数项目）：

- `*.proto` `*_proto.*`
- `**/architecture/*` `**/arch/*`
- `**/middleware/*`
- 项目根的 `Makefile` `Dockerfile` `go.mod` `package.json` `pyproject.toml` `Cargo.toml` `pom.xml`
- `**/SKILL.md` `.claude/skills/**`
- `config*.yaml` `config*.yml`

命中任何一个就标记 `arch_changed=true`。

### 5. 更新 Obsidian activeContext.md

打开 `$VAULT_PATH/<obsidian_project_path>/activeContext.md`，在 `## 最近改了什么（最近 1-2 次会话）` 区域**顶部插入**一行：

```
- <YYYY-MM-DD> `<short_sha>` <commit_message>
```

**规则**：

- 该区域只保留最新 5 条，更早的删掉
- 如果最新 sha 已存在，跳过（防重复）
- 如果区域不存在，提示用户文件结构异常，让其检查模板

### 6. 更新首页项目活跃状态

打开 `$VAULT_PATH/首页.md`，找到"项目列表"表格里 obsidian 路径匹配当前项目的那一行，把"状态"列更新为：

```
进行中（YYYY-MM-DD）
```

**只动这一个 cell**，不要改其他列。如果该列原本是「维护中」「已暂停」等用户手填状态，跳过更新（人工设置优先）—— 仅当原值是「进行中」或为空时才覆盖。

### 7. 检查 CLAUDE.md 是否有需要迁移的旧结构

v0.5 以后，"最近变更"由 `activeContext.md` 承载，CLAUDE.md 不再有此段。

- 如果 `$CODE_ROOT/CLAUDE.md` 仍包含 `## 最近变更` 段 → 输出提醒：
  ```
  ⚠️ 检测到旧版 CLAUDE.md 结构（含"最近变更"段）。
     建议用 /update-memory 迁移到 v0.5 新骨架，将"最近变更"移到 activeContext.md。
     本次 /sync-docs 仍会更新该段，但后续请迁移。
  ```
  并继续向旧 `## 最近变更` 写入（向下兼容，防已有文件断更）
- 如果没有 `## 最近变更` 段（已是 v0.5 结构）→ 跳过，什么都不写

### 8. 刷新 CLAUDE.md 的"Obsidian 文档地图"段

**目的**：让 Claude 下次开工前不用 Read 概览/首页就能知道 Obsidian 里有哪些子文档。

**执行**：

1. 读 `$VAULT_PATH/<obsidian_project_path>/概览.md` 的 frontmatter `docs:` 字段
   - 这是人工维护的真相源（含 `file:` 和 `desc:`）
2. `find $VAULT_PATH/<obsidian_project_path>/ -name "*.md" -type f` 扫实际文件
3. **合并策略**：
   - frontmatter 里有的：用 frontmatter 的 `desc`
   - 实际存在但 frontmatter 没列的：追加，desc 留 `_(待补充说明，可在 概览.md 的 docs: 字段添加)_`
   - frontmatter 列了但文件不存在的：标 `⚠️ 文件不存在，请检查` 并保留（提示用户清理）
4. 按目录分组生成新的"Obsidian 文档地图"段内容（格式见模板）
5. **diff 现有 CLAUDE.md 的"## Obsidian 文档地图"段**：
   - 内容相同 → **跳过写入**（保护 prefix cache，不无谓改动）
   - 内容不同 → 用 Edit 替换该段
6. 没有这段的旧版 CLAUDE.md → **跳过并报告**："CLAUDE.md 缺少 Obsidian 文档地图段，请用新模板迁移或运行 /update-memory"

### 9. 输出报告

```
✅ 已同步 <obsidian_project_path>
  - activeContext.md ← +1 条提交
  - 首页.md ← 项目状态已刷新
  - CLAUDE.md ← 最近变更 +1 条 / 文档地图 [无变化 | 刷新 +N -M]
  ⚠️ 检测到架构变更：<file1>, <file2>
     建议手动检查 $VAULT_PATH/<obsidian_project_path>/01-架构/ 是否需要更新
  ⚠️ frontmatter docs: 列了 X 个文件不存在：<list>
     建议清理 概览.md 的 docs: 字段
```

## 注意事项

- **只动机器维护区**：`activeContext.md` 的"最近改了什么"、`首页.md` 的项目状态列、CLAUDE.md 的"最近变更"。其他人写的内容不要碰。
- **跨仓库依赖联动**：如果当前项目在 `$VAULT/CLAUDE.md` 的"跨仓库联动规则"段被列为"上游共享库"，提交后额外提示用户去下游项目的 `activeContext.md` 加"待联动"条。
- **merge commit 跳过**：用 `git log --no-merges -1` 时如果命中 merge commit，回退到上一个非 merge commit。
