---
description: 智能 git pull：拉取最新代码、分析变更、调用 /sync-docs 同步到 Obsidian。
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion]
---

# /pull — 智能拉取 + 文档同步

封装 `git pull`，拉取后分析新提交、调用 [`sync-docs`](./sync-docs.md) 把所有新增提交批量同步到 Obsidian。

## 执行流程

### 1. 检查工作区干净

```bash
git status --short
```

如果有未提交的改动，停下来用 AskUserQuestion 问用户：

- 暂存（`git stash`）后再 pull
- 取消，让用户先 commit
- 强制继续（不推荐，会触发 merge 冲突）

### 2. 记录拉取前位置

```bash
BEFORE_SHA=$(git rev-parse HEAD)
BRANCH=$(git branch --show-current)
```

### 3. 执行 pull

```bash
git pull --ff-only origin "$BRANCH" 2>&1
```

**优先用 `--ff-only`** 避免意外的 merge commit。如果失败（远端有不能 fast-forward 的提交），停下来报告：

```
⚠️ 远端有分叉提交，无法 fast-forward。
   选择：
   1. git pull --rebase origin <branch>（保持线性历史）
   2. git pull --no-ff origin <branch>（保留 merge commit）
   3. 取消，手工解决
```

用 AskUserQuestion 让用户选。

如果有冲突，报告冲突文件并停止，**不做文档同步**。

### 4. 检查是否有更新

```bash
AFTER_SHA=$(git rev-parse HEAD)
```

如果 `BEFORE_SHA == AFTER_SHA`，输出"已是最新，无新提交"并退出。

### 5. 分析新增提交

```bash
git log --no-merges --reverse --format="%h|%s|%ai|%an" "$BEFORE_SHA..$AFTER_SHA"
git diff --name-only "$BEFORE_SHA..$AFTER_SHA"
```

`--no-merges` 跳过 merge commit，只记真实改动。

### 6. 检测架构变更

复用 [`sync-docs`](./sync-docs.md) 第 4 步的文件模式匹配规则。命中任意一个就标记 `arch_changed=true`。

### 7. 批量同步到 Obsidian

调用 [`sync-docs`](./sync-docs.md) 的核心逻辑，但有两点不同：

- **批量插入**：把 `BEFORE_SHA..AFTER_SHA` 之间的所有非 merge 提交一次性写入 `activeContext.md` 的"最近改了什么"，每条加 `(pull)` 后缀区分来源：
  ```
  - 2026-04-19 `a3f5c1d` 修复订单状态机的并发问题 (pull)
  ```
- **裁剪**：保留最新 5 条
- **超过 5 条提示**：如果新增提交数 > 5，只插入最近 5 条，输出附带提示：

  ```
  本次 pull 共 N 条提交，仅记录最新 5 条到 activeContext。
  完整列表见 git log。
  ```

### 8. 更新首页项目状态

跟 [`sync-docs`](./sync-docs.md) 第 6 步一样，把首页表格里当前项目的"状态"列刷成 `进行中（YYYY-MM-DD）`。

### 9. 输出报告

```
✅ git pull 完成
   <BEFORE_SHA> → <AFTER_SHA>
   分支：<branch>
   新增提交：N 条（已同步最新 5 条到 activeContext）
   
✅ 文档已同步：<obsidian_project_path>
   ⚠️ 检测到架构变更：<file1>, <file2>
      建议手动检查 01-架构/ 文档
```

## 注意事项

- **不要在 pull 失败时同步文档**：冲突 / 网络问题 / fast-forward 失败都要停下来报告，不要带着脏状态去更新 vault
- **跨仓库联动**：pull 拉到了上游共享库（如 proto 仓库）的变更时，提示用户去下游项目的 `activeContext.md` 加"上游已变更，待联动验证"条
- **stash 恢复**：如果第 1 步用了 `git stash`，pull 完成后**自动 `git stash pop`**，并在输出里说明
