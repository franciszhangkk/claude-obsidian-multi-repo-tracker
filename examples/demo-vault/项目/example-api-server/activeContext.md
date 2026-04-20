# example-api-server - 当前焦点

> **每次开工前必读。** 这个文件应该回答："我现在在做什么？最近改了什么？下一步是什么？"

> ⚠️ **不是流水账**。只保留"当前状态"，旧内容移到 progress.md。建议长度不超过一屏。

## 当前焦点

把订单状态机从 if/else 重构成 state machine 库，支持后续加新状态。

## 最近改了什么（最近 1-2 次会话）

- 抽出 `internal/service/order/state/` 子包，引入 `looplab/fsm` 库
- 加了 8 个状态转换的单测
- proto 已变更，待联动验证（[[项目/example-shared-proto/activeContext]] 新增了 `OrderRefunded` 状态）

## 下一步

- [ ] 把现有 `OrderService.UpdateStatus` 的 if/else 全部替换成 fsm 调用
- [ ] 验证 proto 新增的 `OrderRefunded` 状态在 fsm 里能正确流转
- [ ] 联动通知前端（[[项目/example-web-app/activeContext]]）

## 阻塞 / 待决问题

- 历史订单库里有 200+ 条状态非法的脏数据，重构前要不要先清洗？等产品决策

## 涉及文件

- `internal/service/order/state/machine.go`
- `internal/service/order/state/machine_test.go`
- `internal/service/order/service.go`（待改）

---

> 更新方式：`/update-active` 命令，或手动编辑。完成的事项移到 [[progress]]。
