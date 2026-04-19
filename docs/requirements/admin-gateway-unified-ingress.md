# Admin Gateway 统一入口基线

## Scope

- scope: 为 admin 端补齐 Gateway 统一入口能力，把本地联调和后续部署的 admin API 入口从直连 Admin BFF 收敛到 API Gateway。
- affected users: Admin Web 联调人员、平台实施人员、超级管理员、机构管理员。
- changed behavior: Gateway 新增 `/api/admin/**` 透传入口；admin 前端的 server route handlers 默认改为通过 Gateway 访问 admin API，而不是直连 `5146` 的 Admin BFF。
- dependent systems: `nursing-backend-services` 的 API Gateway、Admin BFF、Identity、Tenant；`nursing-admin-v2` 的 Next route handlers、`.env.local`、本地登录与 live 页面读取路径。
- verification: nursing-documents `npm run docs:build`；nursing-backend-services `dotnet build nursing-backend-services.slnx`；nursing-admin-v2 `npm run lint`、`npm run build`；本地 smoke 验证 `5200/api/gateway/bootstrap` 与多条 `5200/api/admin/**` 返回 200。
- rollback: 回退 Gateway `/api/admin/**` 代理；把 admin 前端入口恢复为直连 `5146`；保留现有页面路径与调用 DTO，不做页面级回滚。

## 背景

- 当前 Gateway 只提供 `/api/gateway/bootstrap`，admin web 的 Next route handlers 默认直连 Admin BFF `5146`。
- 这会导致本地联调和后续部署存在双入口：平台上下文走 Gateway，业务 API 走 Admin BFF，边界不一致，排障和发布都不稳定。
- 本次目标不是重写 admin 页面，也不是改业务契约，而是补齐统一 ingress，使 admin 端具备单入口联调和可观测性基线。

## 目标

- 让 admin 前端在本地和后续环境都优先通过 Gateway 访问 `/api/admin/**`。
- 保持现有 Admin BFF 契约、返回结构和页面状态机不变，避免额外业务回归。
- 为后续网关层统一认证、限流、审计和灰度留出稳定承载面。

## 用户影响

- 登录后进入首页、老人、评定、员工、房间、运营、财务等 live 页面时，页面可见行为不应变化。
- 加载态、空态、错误态仍沿用当前页面实现；差异只在入口从直连 BFF 切到 Gateway。
- 当 Gateway 正常而 Admin BFF 不可达时，用户现在会得到更准确的“统一入口失败”信号，而不是前端隐式绕过网关。

## 成功判据

- `GET /api/gateway/bootstrap` 继续返回 200。
- `GET /api/admin/assessments`、`/api/admin/health/archives`、`/api/admin/staff`、`/api/admin/organizations`、`/api/admin/rooms`、`/api/admin/activities`、`/api/admin/incidents`、`/api/admin/equipment`、`/api/admin/supplies`、`/api/admin/finance/summary` 通过 Gateway 返回与直连 Admin BFF 一致的 200 / 业务载荷。
- admin 前端本地启动后，关键 live 页面无需修改页面路径即可继续正常读取数据。

## 非目标

- 不在本次把 Family / Nani 的业务 API 也统一收口到 Gateway。
- 不在本次引入 YARP 或新的网关依赖。
- 不在本次调整 Admin BFF 路由命名、DTO 或页面渲染结构。

## 风险与回滚

- 若 Gateway 代理逻辑出现路径或头透传错误，admin 页面会在 live 模式下统一失败，影响面比直连 BFF 更集中。
- 回滚方式必须保持简单：删除 Gateway admin catch-all 代理，并把 admin 前端环境变量恢复到 `http://localhost:5146`。
- 若仅前端入口切换有问题，可先只回退 admin 前端服务地址，不必回退 Gateway bootstrap。