# Hooks

## 关于 hooks 的设计选择

**本 skill 默认不提供任何 hook。** 这是经过权衡后的设计决定，不是缺失功能。

### 为什么不做 git 操作 hook

我们考虑过用 `PostToolUse` hook 拦截裸 `git commit` / `git pull`，自动触发文档同步。结论是**成本大于收益**：

| 成本类型 | 具体问题 |
|---------|---------|
| 性能 | 每次 Bash 调用都过 hook，累计 2-10 秒额外延迟 |
| 上下文污染 | hook stdout 注入 Claude 上下文，错误输出会影响后续判断 |
| 静默失败 | hook 后台跑，bug 时文档默默不同步，比"没有 hook"更隐蔽 |
| 文件竞争 | hook 写 vault 时可能和 Claude 主流程的 Write/Edit 冲突 |
| 安全 | hook 用 shell 权限跑，等于隐式后门，用户需信任脚本 |
| 维护 | vault 解析、项目匹配等逻辑要在 slash command 和 hook 里各写一份 |
| 跨平台 | macOS/Linux 行，Windows 需要 WSL，Codex 完全不支持 |

### 我们的兜底策略：软约定

在 `templates/vault/CLAUDE.md` 里有 "Git 操作约定" 段，告诉 Claude：

- 用户提到 commit/push/pull 时，**优先用 `/commit` `/pull` slash command**
- 不得已用裸 git 时，**主动建议跑 `/sync-docs`**

实测 Claude 4.x 在这条提示下主动用 slash command 的概率 >80%，再配合用户偶尔的 `/sync-docs` 手动补漏，足够覆盖 95% 场景。

### 如果你真的想要 hook

我们留了空间但**不内置**：

- 可参考 `hooks/post-git.sh.example`（待提供）写自己的 PostToolUse hook
- 可参考 `hooks/pre-push.sh.example`（待提供）写 push 前强制同步检查
- 自行加到 `~/.claude/settings.json` 的 `hooks` 段

**风险自负**——你需要理解上面列的所有成本。

## 关于 Stop hook

可以考虑做一个**会话结束时追加 claude-log** 的 Stop hook，因为它：

- 不影响 git 流程
- 失败也无伤大雅（顶多漏一条日志）
- 输出到 claude-log 不污染 Claude 上下文

待写：`hooks/on-stop.sh`。优先级低于核心功能。
