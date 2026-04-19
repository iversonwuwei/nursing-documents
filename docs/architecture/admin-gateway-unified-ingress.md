# Admin Gateway 统一入口设计

## Scope

- scope: 设计 API Gateway 对 `/api/admin/**` 的统一透传，以及 admin Next server route handlers 从直连 Admin BFF 切换到 Gateway 的实现策略。
- boundaries: 覆盖 Gateway 路由、下游服务地址解析、认证头透传、admin 前端代理基址切换；不改 Admin BFF 业务处理逻辑，不改页面组件。
- dependencies: API Gateway `Program.cs`、Gateway `appsettings.json`、Admin BFF 本地端口 `5146`、admin server proxy helpers、NextAuth session/JWT、Identity/tenant 平台上下文。
- failure modes: Gateway 忘记透传 query 或 Authorization 导致 401/404；路径被错误截断导致新接口丢失；前端仍保留 `5146` 直连导致联调绕过 Gateway；旧 Next 进程未重启导致环境变量仍指向旧入口。
- verification: nursing-documents `npm run docs:build`；backend `dotnet build`；admin `npm run lint` 与 `npm run build`；浏览器或脚本验证关键 admin API 通过 `5200` 返回 200。
- rollback: 删除 Gateway `/api/admin/{**path}` 代理并恢复前端入口到 `5146`。

## 设计策略

- Gateway 继续保留现有 `/api/gateway/bootstrap` 聚合入口。
- Gateway 新增一个最小 catch-all 代理：`/api/admin/{**path}`，只负责把请求无状态透传到 Admin BFF。
- Admin 前端不改页面层请求路径，仍由现有 Next route handlers 调用 `forwardToAdminBff` / `forwardWorkflowRequest`；仅把它们的上游基址从 Admin BFF 改为 Gateway。

## 请求流

1. 浏览器访问 admin 页面。
2. 页面调用现有 Next route handler，例如 `/api/assessments`、`/api/staff`、`/api/admin-operations/*`。
3. Next server route handler 继续从 session / JWT 取 access token 和 tenant，并向统一入口 `5200` 发起请求。
4. Gateway 收到 `/api/admin/**` 后把 Authorization、tenant、correlation 等头和 query 原样转发给 Admin BFF `5146`。
5. Admin BFF 继续向下游领域服务聚合并返回原有 DTO。

## 关键设计点

### 1. 契约兼容

- Gateway 不重塑响应体，不重新包装 ProblemDetails。
- Gateway 仅复制下游状态码和内容类型，保证 admin 前端当前解析逻辑不需要改。
- `/api/admin/**` 的 URL 形态保持不变，前端只改基址，不改路径拼接规则。

### 2. 认证与租户上下文

- Gateway 不自行重新签发 admin token，只透传来自 admin Next proxy 的 Authorization。
- 继续透传 `X-Tenant-Id` 和 `X-Correlation-Id`，保证现有后端上下文与观测链路可用。
- `RequireAuthorization()` 仍挂在 Gateway admin 代理入口上，避免未登录请求绕过网关直接打 BFF。

### 3. 发布与回滚

- 这是可逆改动：先落网关代理，再切前端入口。
- 若后端代理通过但前端不稳定，可以仅回退前端环境变量到 `5146`，网关代理代码可先保留。
- 若网关代理本身有问题，可以删除 catch-all 代理，admin 继续沿用当前直连 BFF 方式。

## 健康信号

- Gateway readiness 正常且 `GET /api/gateway/bootstrap` 为 200。
- 多个 Admin live 读接口通过 `5200` 返回 200，且响应体字段与 `5146` 一致。
- admin 前端登录后，首页与关键 live 页面不出现新的 `503` 或统一 404。

## 残余风险

- 当前 Gateway 仅承担简单透传，尚未做细粒度路由治理、限流、熔断和审计分类。
- 由于 admin 前端大量 Next route handlers 共享同一服务基址，若忘记重启 dev server，可能继续命中旧入口并造成误判。
- 本次只统一 admin 入口；Family/Nani 仍保留各自入口，整体 edge 还不是完全单入口。