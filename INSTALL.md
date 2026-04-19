# 安装指南

## 前置条件

- Claude Code（主要使用环境）
- Obsidian（可选 —— 没装也能用，纯 markdown 文件直接编辑）
- Git
- macOS / Linux / Windows + WSL

## 安装步骤

### 1. Clone 到 Claude Code skills 目录

```bash
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git \
  ~/.claude/skills/claude-obsidian-multi-repo-tracker
```

### 1.5 注册 slash commands

Claude Code 的 slash command 从 `~/.claude/commands/` 加载。把本 skill 的 `commands/` 软链过去：

```bash
# 软链整个目录（推荐，升级时自动同步）
mkdir -p ~/.claude/commands
ln -sf ~/.claude/skills/claude-obsidian-multi-repo-tracker/commands/init-vault.md \
       ~/.claude/commands/init-vault.md
ln -sf ~/.claude/skills/claude-obsidian-multi-repo-tracker/commands/add-project.md \
       ~/.claude/commands/add-project.md
# 其他命令同理
```

或者直接复制（不会随 skill 升级而更新）：

```bash
cp ~/.claude/skills/claude-obsidian-multi-repo-tracker/commands/*.md ~/.claude/commands/
```

### 2. 配置 vault 路径

在 `~/.claude/settings.json` 添加：

```json
{
  "env": {
    "OBSIDIAN_VAULT_PATH": "/path/to/your/Obsidian Vault"
  }
}
```

或者临时：

```bash
export OBSIDIAN_VAULT_PATH="$HOME/Desktop/Obsidian Vault"
```

### 3. （可选）开启 Stop hook 自动写日志

在 `~/.claude/settings.json` 加 hook：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/skills/claude-obsidian-multi-repo-tracker/hooks/on-stop.sh"
          }
        ]
      }
    ]
  }
}
```

### 4. 重启 Claude Code

```bash
# 退出当前会话，重新启动
claude
```

## 验证

在 Claude Code 里运行：

```
/init-vault
```

应该看到向导提示输入 vault 路径。

## 升级

```bash
cd ~/.claude/skills/claude-obsidian-multi-repo-tracker
git pull
```

## 卸载

```bash
rm -rf ~/.claude/skills/claude-obsidian-multi-repo-tracker
# 删除 settings.json 里的 OBSIDIAN_VAULT_PATH 和 hooks
```

vault 里的文档**不会被删除**，因为它们是你的内容。
