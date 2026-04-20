# example-api-server 认证流程

> 一个深度文档示范。展示「Obsidian 替代代码扫描」的价值 — 读这一篇就能给同事讲清认证链路。

## 链路总览

```
前端 → API Gateway → example-api-server
                     ├── JWT 校验中间件 (internal/middleware/auth.go)
                     ├── 业务 handler (internal/api/order/)
                     └── 调用其他服务时携带 X-User-Id（gRPC metadata）
```

## JWT 结构

```json
{
  "uid": "user_xxx",
  "tenant": "tenant_yyy",
  "exp": 1234567890,
  "scope": ["order.read", "order.write"]
}
```

签名算法 RS256，公钥从配置中心拉取，**5 分钟刷新一次**（`internal/auth/key_rotator.go`）。

## 关键代码位置

- 中间件：`internal/middleware/auth.go:42`
- Token 解析：`internal/auth/jwt.go:88`
- 公钥轮换：`internal/auth/key_rotator.go:120`
- Scope 校验：`internal/auth/scope.go:30`

## 设计决策

### 为什么不直接信 API Gateway 的认证？

Gateway 只做粗粒度限流和黑名单，**业务级权限（scope）必须在服务内校验**，否则一旦 Gateway 配置漏改就会越权。

### 为什么 5 分钟刷新公钥？

平衡安全和性能：太长则密钥泄漏后窗口大，太短则配置中心压力大。Auth 团队约定 4 小时轮换密钥，5 分钟刷新足够。

## 已知坑

- 时钟漂移：服务器时间和签发方差超过 30 秒就会拒绝 token，K8s 节点要装 chrony
- Token 长度：含 scope 后达到 1.2KB，**header 单独限制要调到 8KB**

## 联动

涉及 proto 字段时看 [[项目/example-shared-proto/概览]]。
