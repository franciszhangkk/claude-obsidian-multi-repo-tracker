# example-api-server - 进度

> 项目状态摘要。**不是 changelog**，是"现在能用啥、还差啥"。

## 已完成（能用的功能）

- 订单 CRUD 接口（HTTP + gRPC 双协议）
- 订单状态流转：`Created → Paid → Shipped → Completed`
- Redis 缓存层（订单详情 5 分钟 TTL）
- Prometheus 指标 + 结构化日志

## 进行中

- 状态机重构（详见 [[activeContext]]）
- 退款流程接入

## 待办（已规划但没开始）

- 订单分库分表（DAU > 10w 后）
- 异步事件持久化到 Kafka（替代 Redis Pub/Sub）

## 已知问题

- 高并发下 Redis 缓存击穿：单热点订单查询 QPS > 5k 时偶发数据库压力突增
- gRPC 连接池在 K8s 滚动更新时存在 1-2 秒的请求失败

## 决策记录

| 日期 | 决策 | 原因 |
|------|------|------|
| 2026-03-15 | 状态机用 `looplab/fsm` 而非自己写 | 维护成本低，社区验证过 |
| 2026-03-02 | Redis 而非 Memcached | 团队已有运维经验，且需要 Pub/Sub |
| 2026-02-10 | 订单 ID 用雪花 ID 而非 UUID | 更短、可排序、利于分库 |

---

> 更新方式：完成里程碑时手动编辑，或 `/weekly-digest` 自动汇总日记。
