# example-web-app - 当前焦点

> **每次开工前必读。** 这个文件应该回答："我现在在做什么？最近改了什么？下一步是什么？"

## 当前焦点

接入退款流程：新增退款申请页 + 退款进度查询。

## 最近改了什么（最近 1-2 次会话）

- 升级 grpc-web 到 1.5
- 把订单详情页的 polling 换成 server-streaming（依赖后端先支持）
- proto 已变更，待联动验证（[[项目/example-shared-proto/activeContext]] 新增 `OrderRefunded`）

## 下一步

- [ ] 新增 `src/features/order/RefundRequest.tsx` 页面
- [ ] 在订单状态枚举里加 `OrderRefunded`，更新 i18n 文案
- [ ] 等 [[项目/example-api-server/activeContext]] 后端接口就绪后联调

## 阻塞 / 待决问题

- 退款 UI 设计稿还没出，目前只能搭骨架

## 涉及文件

- `src/features/order/`（新建 RefundRequest.tsx）
- `src/api/order.ts`（加 refund 相关 hooks）
- `src/i18n/zh-CN/order.json`（新增退款文案）

---

> 更新方式：`/update-active` 命令，或手动编辑。完成的事项移到 [[progress]]。
