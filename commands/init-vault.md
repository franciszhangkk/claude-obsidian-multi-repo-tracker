---
description: 初始化一个 Obsidian vault 作为 Claude Code 的多项目跟踪知识库
allowed-tools: [Bash, Read, Write, Edit, Glob, AskUserQuestion]
---

# /init-vault — 初始化 Obsidian vault

你的任务：把 `templates/vault/` 下的模板复制到用户指定的 vault 路径，建立"多项目跟踪知识库"的基础结构。

## 执行流程

### 1. 确定 vault 路径

按这个优先级找：

1. 用户在命令后传的路径参数（例如 `/init-vault ~/Documents/MyVault`）
2. 环境变量 `$OBSIDIAN_VAULT_PATH`
3. 都没有 → 用 AskUserQuestion 问用户，给出常见候选：
   - `~/Documents/Obsidian Vault`
   - `~/Desktop/Obsidian Vault`
   - 让用户输入自定义路径

### 2. 检查目标路径状态

```bash
ls -la "<vault_path>" 2>&1
```

- **不存在** → 直接创建
- **存在但为空** → 直接初始化
- **存在且非空** → 用 AskUserQuestion 问用户：
  - 在现有 vault 上叠加（仅写不存在的文件，已有的不覆盖）
  - 换个路径
  - 取消

### 3. 找到本 skill 的安装路径

```bash
# 通常是 ~/.claude/skills/claude-obsidian-multi-repo-tracker/
# 通过查找 templates/vault/首页.md 验证
SKILL_DIR="$HOME/.claude/skills/claude-obsidian-multi-repo-tracker"
[ -f "$SKILL_DIR/templates/vault/首页.md" ] || echo "skill 安装位置异常，请检查"
```

### 4. 创建目录结构

```bash
mkdir -p "<vault>/项目"
mkdir -p "<vault>/日记/claude-log"
mkdir -p "<vault>/教程"
mkdir -p "<vault>/学习笔记"
```

### 5. 复制 vault 模板

将 `$SKILL_DIR/templates/vault/` 下的四个文件复制到 vault 根目录：

- `首页.md`
- `开发工作流指南.md`
- `CLAUDE.md`
- `AGENTS.md`（用户问起再说，默认也复制——不影响 Claude Code）

**重要**：
- 如果目标文件已存在，**不要覆盖**，跳过并在最后报告
- 复制后用 Read 检查是否有 `{{占位符}}`，目前模板里没有但未来可能加

### 6. 设置环境变量提醒

提醒用户在 `~/.claude/settings.json` 加：

```json
{
  "env": {
    "OBSIDIAN_VAULT_PATH": "<刚才用户选的 vault 路径>"
  }
}
```

或者临时（当前 shell）：

```bash
export OBSIDIAN_VAULT_PATH="<vault_path>"
```

### 7. 输出总结

格式：

```
✅ Vault 初始化完成

路径：<vault_path>
新建文件：
  - 首页.md
  - 开发工作流指南.md
  - CLAUDE.md
  - AGENTS.md
新建目录：
  - 项目/
  - 日记/claude-log/
  - 教程/
  - 学习笔记/
跳过（已存在）：
  - <如有>

下一步：
  - 设置 $OBSIDIAN_VAULT_PATH 环境变量（见上）
  - 运行 /add-project <项目名> <代码路径> 添加你的第一个项目
```

## 错误处理

- 路径不可写 → 报错，建议用户检查权限
- skill 安装路径找不到模板 → 报错，告诉用户重新 clone
- 用户中途取消 → 友好退出，不留半成品（如果创建了空目录就删掉）

## 注意

- 不要硬编码 frank 的路径（`~/Desktop/Obsidian Vault`），那只是默认候选之一
- 不要绑定语言或技术栈
- 不要假设用户已经在用 Obsidian 应用——纯文件操作，Obsidian 装不装都能跑
