# Codex 兼容性

## TL;DR

- ✅ vault 结构 + AGENTS.md：Codex 可用
- ❌ commands / hooks：Claude Code 专属

## Codex 用户怎么用

### 1. 复制 vault 模板

```bash
cp -r templates/vault/* /path/to/your/vault/
```

### 2. 把 AGENTS.md 放到 vault 根

```bash
cp templates/vault/AGENTS.md /path/to/your/vault/AGENTS.md
# 同时也可保留 CLAUDE.md，不影响
```

### 3. 复制项目模板

每加一个项目，手动复制：

```bash
mkdir -p /path/to/vault/项目/<name>
cp -r templates/project/* /path/to/vault/项目/<name>/
```

### 4. 文档同步靠手动

Codex 没有 `/sync-docs` `/commit` `/pull` 命令。可以：

- 改完代码后，让 Codex 帮你写一段更新 `activeContext.md`
- 或者写一个 shell 脚本封装 git diff → 文档更新逻辑

## 能力对照

| 能力 | Claude Code | Codex |
|------|:-----------:|:-----:|
| 自动加载 vault 指令 | CLAUDE.md ✅ | AGENTS.md ✅ |
| 跨仓库依赖图 | ✅ | ✅ |
| `/sync-docs` 等命令 | ✅ | ❌ |
| Stop hook 自动写日志 | ✅ | ❌ |
| `/init-vault` 向导 | ✅ | ❌（手动复制模板） |

## 未来计划

如果 Codex 发布了类似 skill / hook 的扩展机制，会考虑提供 Codex 版命令。当前 Codex 的扩展能力有限，**不做兼容性妥协**——主目标始终是 Claude Code。
