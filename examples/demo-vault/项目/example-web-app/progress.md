# example-web-app - 进度

> 项目状态摘要。**不是 changelog**，是"现在能用啥、还差啥"。

## 已完成（能用的功能）

- 订单列表 / 详情页
- 下单流程（含支付跳转）
- 物流查询
- 多语言支持（zh-CN / en-US）

## 进行中

- 退款流程（详见 [[activeContext]]）

## 待办（已规划但没开始）

- PWA 离线支持
- 订单导出 CSV
- 暗色主题

## 已知问题

- iOS Safari 上 server-streaming 偶发断流
- 弱网下下单按钮可能重复触发，需要加幂等 key

## 决策记录

| 日期 | 决策 | 原因 |
|------|------|------|
| 2026-03-20 | 用 grpc-web 而非 REST | 与后端共享 proto，类型安全 |
| 2026-02-25 | TanStack Query 而非 Redux Toolkit | 服务端状态管理足够，不需要客户端 store |
| 2026-01-15 | Vite 而非 Next.js | 纯 SPA 不需要 SSR |

---

> 更新方式：完成里程碑时手动编辑，或 `/weekly-digest` 自动汇总日记。
