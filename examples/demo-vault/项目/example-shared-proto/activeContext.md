# example-shared-proto - 当前焦点

> **每次开工前必读。** 这个文件应该回答："我现在在做什么？最近改了什么？下一步是什么？"

## 当前焦点

为退款流程加 proto 字段：新增 `OrderRefunded` 状态 + 退款相关 RPC。

## 最近改了什么（最近 1-2 次会话）

- `order/v1/order.proto` 加了 `OrderStatus.ORDER_REFUNDED = 5`
- 新增 `RefundService`：`CreateRefund` / `GetRefund` / `ListRefunds` 三个 RPC
- 已发布 v1.8.0（向后兼容，仅追加字段）

## 下一步

- [ ] 通知下游联动验证
  - [ ] [[项目/example-api-server/activeContext]] — 实现 `RefundService`
  - [ ] [[项目/example-web-app/activeContext]] — TS 类型已生成，前端可用
- [ ] 补 `RefundReason` 枚举（产品还在确认枚举值）

## 阻塞 / 待决问题

- 退款原因枚举待产品确认，目前留 string 占位

## 涉及文件

- `order/v1/order.proto`
- `payment/v1/refund.proto`（新建）

---

> 更新方式：`/update-active` 命令，或手动编辑。完成的事项移到 [[progress]]。
