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

### 7. 同步项目 repo 的 CLAUDE.md（可选）

如果 `$CODE_ROOT/CLAUDE.md` 存在且包含 `## 最近变更`（带或不带 `<!-- 由 /sync-docs 自动维护 -->` 标记）的区域：

- 在该区域顶部插入同样格式的提交记录，保留最近 5 条
- 没有这个区域就**跳过**，不强行创建

### 8. 输出报告

```
✅ 已同步 <obsidian_project_path>
  - activeContext.md ← +1 条提交
  - 首页.md ← 项目状态已刷新
  - CLAUDE.md ← 同步 / 跳过（无该区域）
  ⚠️ 检测到架构变更：<file1>, <file2>
     建议手动检查 $VAULT_PATH/<obsidian_project_path>/01-架构/ 是否需要更新
```

## 注意事项

- **只动机器维护区**：`activeContext.md` 的"最近改了什么"、`首页.md` 的项目状态列、CLAUDE.md 的"最近变更"。其他人写的内容不要碰。
- **跨仓库依赖联动**：如果当前项目在 `$VAULT/CLAUDE.md` 的"跨仓库联动规则"段被列为"上游共享库"，提交后额外提示用户去下游项目的 `activeContext.md` 加"待联动"条。
- **merge commit 跳过**：用 `git log --no-merges -1` 时如果命中 merge commit，回退到上一个非 merge commit。
