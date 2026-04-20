# example-shared-proto - 进度

> 项目状态摘要。**不是 changelog**，是"现在能用啥、还差啥"。

## 已完成（能用的功能）

- 订单 / 支付 / 用户 三个业务域 v1 接口
- buf CI：lint、break change 检测、双语言代码生成
- Go 包发布到内部 module proxy
- TS 包发布到内部 npm registry

## 进行中

- 退款 RPC（详见 [[activeContext]]）

## 待办（已规划但没开始）

- v2 引入 streaming RPC（订单状态变更推送）
- 加 GraphQL gateway（评估中）

## 已知问题

- 无

## 决策记录

| 日期 | 决策 | 原因 |
|------|------|------|
| 2026-03-01 | break change 一律走 v2 包，不在 v1 内改 | 避免下游被迫升级 |
| 2026-02-05 | buf 而非 protoc 直接调 | lint + break change 检测开箱即用 |
| 2026-01-10 | 双语言（Go + TS）由 CI 生成而非手工 | 杜绝版本漂移 |

---

> 更新方式：完成里程碑时手动编辑，或 `/weekly-digest` 自动汇总日记。
