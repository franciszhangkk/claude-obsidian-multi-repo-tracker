# Demo Vault

> **5 分钟看明白这套工作流长啥样。**

这是一个脱敏后的示范 vault，包含 3 个虚构项目：

- `example-api-server`（Go 后端）
- `example-web-app`（前端）
- `example-shared-proto`（gRPC proto 共享库，被前两者依赖）

## 怎么读这个 demo

按以下顺序看，一次过完：

1. **[首页.md](./首页.md)** — 看跨项目入口和依赖关系图
2. **[CLAUDE.md](./CLAUDE.md)** — 看给 Claude 的全局指令长啥样
3. **[项目/example-api-server/概览.md](./项目/example-api-server/概览.md)** — 看项目入口文档
4. **[项目/example-api-server/activeContext.md](./项目/example-api-server/activeContext.md)** — 看"当前焦点"是怎么写的（每次开工必读）
5. **[项目/example-api-server/progress.md](./项目/example-api-server/progress.md)** — 看项目状态摘要
6. **[项目/example-api-server/01-架构/example-api-server-认证流程.md](./项目/example-api-server/01-架构/example-api-server-认证流程.md)** — 看一个深度文档示范
7. **[日记/claude-log/2026-04-19.md](./日记/claude-log/2026-04-19.md)** — 看全局工作日志长啥样

## 关键看点

- **跨仓库依赖**：`首页.md` 的 mermaid 图清晰显示 proto 库被两个服务依赖
- **代码 ↔ 文档对应**：每个项目 `概览.md` 都标注了对应的代码仓库路径
- **三层时间尺度**：`概览`（不变的架构）、`activeContext`（这周在做啥）、`progress`（已经能用啥）
- **跨链接**：所有项目互链都用全路径 `[[项目/<name>/概览]]`，不用裸文件名

## 这只是骨架

每个项目只放了入口三件套 + 一个深度示范，真实使用时 `01-架构/`、`02-功能/`、`05-测试与调试/` 会持续填充。
