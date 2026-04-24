---
description: 智能 git 提交：自动生成 commit message、提交代码、然后调用 /sync-docs 同步到 Obsidian。
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, AskUserQuestion]
---

# /commit — 智能提交 + 文档同步

封装 `git add` + `git commit`，提交完成后自动调用 [`sync-docs`](./sync-docs.md) 把变更同步到 Obsidian。

## 执行流程

### 1. 检查工作区状态

```bash
git status --short
git diff --stat
git diff --cached --stat
```

如果工作区干净（无暂存、无未暂存改动），输出"没有改动需要提交"并退出。

### 2. 选择并暂存文件

读取改动列表，按以下规则筛选要暂存的文件：

- **必须排除**：`.env*`、`*.key`、`*.pem`、`*credentials*`、`*secret*`、`*.p12`、`*.pfx`
- **默认排除**（除非用户明确指定）：`node_modules/`、`__pycache__/`、`*.pyc`、`.DS_Store`、`*.log`
- **可疑文件警告**：文件名含 `password`、`token`、`api_key` 时停下来用 AskUserQuestion 让用户确认

用 `git add <具体文件>` 一个个加，**不要用 `git add .` 或 `git add -A`**。

### 3. 生成 commit message

#### 用户传了消息（`$ARGUMENTS` 非空）

直接使用 `$ARGUMENTS` 作为 commit message，跳到第 4 步。

#### 自动生成

参考最近 5 条提交的风格：

```bash
git log --oneline -5
```

按以下规则生成中文 message：

- 格式：`<type>: <描述>`
- type 选一个：`feat`（新功能）/ `fix`（bug 修复）/ `refactor`（重构）/ `docs`（文档）/ `chore`（杂项）/ `test`（测试）/ `style`（格式）
- 描述用中文，**陈述句**，不超过 50 字
- 不要写"修改了 X 文件"这种废话，要写**为什么**或**做了什么决策**

如果改动横跨多个不相关的功能块，用 AskUserQuestion 问用户是否要拆分提交。

### 4. 执行提交

```bash
git commit -m "$(cat <<'EOF'
<message>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

如果 pre-commit hook 失败：报告错误并停止，**不要用 `--no-verify` 绕过**。

### 5. 调用 sync-docs

提交成功后，**直接执行 [`sync-docs`](./sync-docs.md) 的全部流程**（不要重新解析 vault 路径，复用同一份逻辑）。

如果 sync-docs 失败（vault 不存在、项目未注册等），不要让 commit 一起回滚 —— commit 已经在 git 里了。报告"提交成功，文档同步失败：<原因>"并提示用户后续手动 `/sync-docs`。

### 6. 更新当前焦点（精简版 update-active）

**额外 token 成本约 200**，复用已有的 commit message，不再重新读 git。

从 commit message 推断焦点草稿（和 `/update-active` 第 3 步相同规则，但输入只用 commit message）。

然后用 AskUserQuestion 展示：

```
当前焦点是否需要更新？

  推测："<从 commit message 提取的一句话>"

→ 确认 / 自定义 / s 跳过
```

- 用户确认 → 写入 activeContext.md "当前焦点"段
- 用户跳过 → 不动 activeContext，继续
- 用户自定义 → 写入用户输入的内容

**什么时候建议跳过**：commit type 是 `chore` / `docs` / `style` / `test` 时，推测草稿后主动提示"这次提交不涉及功能进展，建议跳过焦点更新"。

### 7. 输出报告

```
✅ 提交完成
   <short_sha> <message>
   涉及文件: <count> 个
   
✅ 文档已同步（参见 sync-docs 的输出）
```

或：

```
✅ 提交完成
   <short_sha> <message>
   
⚠️ 文档同步失败：<reason>
   稍后可手动运行 /sync-docs 补同步
```

## 注意事项

- **不要 push**：本命令只 commit，push 由用户决定
- **不要 amend**：除非用户用 `$ARGUMENTS` 显式指定 `--amend`
- **不要跳过 hook**：pre-commit/commit-msg hook 失败时停下来给用户看错误，让用户决定怎么处理
