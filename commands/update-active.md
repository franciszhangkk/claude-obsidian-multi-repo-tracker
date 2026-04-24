---
description: 更新 activeContext.md 的"当前焦点"段。从 git 信号推断草稿，用户一键确认，最低成本维护焦点文档。
allowed-tools: [Bash, Read, Write, Edit, AskUserQuestion]
---

# /update-active — 更新当前焦点

**何时调**：切换任务、开始新功能、上次对话结束后。`/commit` 也会自动触发一次精简版。

**设计原则**：agent 产生草稿，用户只做确认或微调——不让用户从零打内容。

## 执行流程

### 1. 定位 vault 和项目

```bash
CODE_ROOT=$(git rev-parse --show-toplevel)
VAULT_PATH="${OBSIDIAN_VAULT_PATH:-}"
```

从 vault CLAUDE.md 映射表找到对应的 `obsidian_project_path`（同 sync-docs 第 2 步）。

找不到时用 AskUserQuestion 让用户选。

### 2. 读取信号（~300 token）

```bash
git log --oneline --no-merges -5      # 最近 commit 在做什么
git status --short                     # 现在改动的是哪些文件
git diff --stat HEAD                   # 未提交改动量级
```

同时读 activeContext.md 现有的"当前焦点"段（了解上一次写的是什么，避免重复）。

### 3. 生成焦点草稿

综合信号推断一句话草稿，格式：

```
<动作>：<具体在做什么>，<阶段>
```

例：
- `Memory 优化：去除幻觉 + 偏好晋升机制，联调测试中`
- `query-material-data skill 拆分：SQL 层完成，接入 skillruntime 中`
- `定时任务 API 联调：ad-plus-media-service ↔ ai-agents gRPC 接口对齐`

**推断规则**：
- 最近 commit type 是 `feat` → 功能开发阶段
- 最近 commit type 是 `fix` → 修复阶段
- 有大量未提交改动（diff 行数 > 100）→ 开发进行中
- 文件路径含 `test` / `_test.go` → 测试阶段
- 同一目录集中改动 → 聚焦该模块

### 4. 展示草稿，等待确认

```
当前焦点建议更新为：

  "Memory 优化：去除幻觉 + 偏好晋升机制，联调测试中"

→ 回车确认 / 输入新内容替换 / s 跳过
```

用 AskUserQuestion，选项：
- `确认（使用以上内容）`
- `跳过（保持不变）`
- `自定义（other 输入）`

### 5. 写入 activeContext.md

用 Edit 把"当前焦点"段内容替换：

```markdown
## 当前焦点

<确认后的内容>
```

**只动这一段**，其他段（最近变更、下一步、阻塞）不动。

### 6. 输出报告

```
✅ 当前焦点已更新
   Memory 优化：去除幻觉 + 偏好晋升机制，联调测试中

   activeContext.md 已同步
```

如果用户跳过：

```
⏭️ 已跳过，当前焦点保持不变
```

## 注意事项

- **只改"当前焦点"段**：不动最近变更（/sync-docs 维护）、不动下一步、不动阻塞
- **草稿推断失败时**：直接让用户输入，不要强行生成无意义草稿
- **已有内容优先**：如果现有焦点和推断结果差不多（关键词重叠 > 50%），不主动建议更新，直接输出 `当前焦点未变更，无需更新`

## 与 /commit 的关系

`/commit` 成功后会自动调用本流程的**精简版**（省略第 2 步读 git，直接用 commit message 推断），token 成本约 200。独立调用 `/update-active` 适合没有 commit 但切换了任务的场景。
