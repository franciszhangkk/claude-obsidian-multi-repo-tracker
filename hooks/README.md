# Hooks

> ⚠️ **MVP 占位**：hooks 是可选增强，第一版可以不做。

## 计划的 hooks

### `on-stop.sh`

在每次 Claude 会话结束时（Stop 事件），自动追加一行到当天的 claude-log：

```
~/<vault>/日记/claude-log/YYYY-MM-DD.md
```

记录内容：时间戳 + 本次会话的工作摘要（让 Claude 自己产出一句话总结）。

### 配置方法

详见 [INSTALL.md](../INSTALL.md) 的 "开启 Stop hook" 章节。

## 设计原则

- hooks 是**可选**的，不开启不影响核心功能
- 不阻塞主流程（异步追加，失败不报错）
- 不读敏感信息（只记摘要，不记完整对话）
