# Admin SaaS 基线设计

## Scope

- scope: 设计 nursing-admin-v2 的 SaaS 前端接入层，包括租户上下文解析、身份源抽象、功能开关模型与 BFF 代理的 token/tenant 传递方式。
- boundaries: 覆盖登录页、NextAuth session/JWT、Next route handlers、顶部导航和 AI 根入口；不重写 backend tenant / identity / billing 服务。
- dependencies: backend identity service、tenant service、api gateway bootstrap、NextAuth、admin BFF 代理、`TopNavbar.tsx`。
- failure modes: 固定 `tenant-demo` 导致跨租户串上下文、前端登录态与代理 token 分裂、某租户禁用 AI 但导航仍暴露入口、platform 模式下 identity 不可达时登录或代理体验不可控。
- verification: nursing-documents `npm run docs:build`，admin `npm run lint`、`npm run build`、聚焦 smoke。
- rollback: 回退 SaaS config helpers、auth route、登录页租户输入、server proxy token 复用与租户开关 UI。

## Design Strategy

- 引入统一 SaaS runtime config，集中管理 `authMode`、默认 tenant、workflow mode、AI mode 和 fallback tenant descriptors。
- 会话模型分两层：
  - JWT token 保存敏感字段，如 platform access token。
  - client-visible session 仅保存 tenant、role、enabled modules/features 和 auth source。
- server route handlers 通过 JWT 读取 access token；若会话中没有真实 token，再退回 dev-login 发行临时 token，保持本地联调兼容。
- tenant service 返回的 `EnabledModules` 和 `EnabledFeatures` 作为第一批 entitlement 源，先落到导航可见性和 AI 根入口禁用态。

## Runtime Model

- `authMode`
  - `demo`: 保持本地 credentials + fallback tenant descriptors。
  - `platform`: 登录时调用 identity service 发行平台 token，并读取 tenant descriptor。
- `tenantContext`
  - 登录表单显式传入 `tenantId`。
  - JWT / session 持久化 `tenantId`、`tenantName`、`plan`、`enabledModules`、`enabledFeatures`。
  - route handlers 优先使用 JWT 中的 `tenantId` 与 `accessToken`。
- `featureFlags`
  - `ai` module 决定 AI 根入口是否在导航中可见。
  - `advanced-audit`、`care-workflow` 等 feature 先进入统一 helper，为后续页面级控制预留单点判断。

## Auth And Tenant Flow

1. 用户在登录页输入用户名、密码和租户。
2. 前端 auth route 校验本地 demo 用户目录，得到 admin / manager 基础身份。
3. 若为 `platform` 模式：
   - 调用 identity service `dev-login` 发行 platform token。
   - 调用 tenant service 获取租户 descriptor。
4. JWT 保存 access token；session 暴露 tenant 与 entitlement。
5. admin route handlers 使用 JWT token 和 session tenant 代理到 downstream。

## Healthy Signals

- 顶部导航可稳定显示租户信息，并在不同租户间切换模块可见性。
- `tenant-private` 等未启用 AI 的租户，导航中不再出现 `AI助手`，直接访问 `/ai-assistant` 时看到显式禁用提示。
- platform 模式下，server proxy 发送的 `X-Tenant-Id` 与会话 tenant 一致，不再固定为 `tenant-demo`。
- demo 模式和 platform 模式都能通过相同的 lint/build/smoke 门禁。

## Verification Strategy

- 文档侧：`npm run docs:build`。
- 静态门禁：`npm run lint`、`npm run build`。
- 浏览器侧至少验证三类信号：
  - 默认租户登录仍可进入首页。
  - tenant-private 登录后 AI 能力被正确隐藏或禁用。
  - scene 链路 smoke 不因租户/身份改造而回归。

## Evolution Rules

- 后续若接入真实 IAM，应复用当前 `authMode` 抽象，而不是再创建第二套登录路由。
- 后续若接入真实订阅与计费，应优先扩展 tenant descriptor / entitlement helper，而不是把套餐判断散落到页面。
- 后续若前端需要 tenant bootstrap，应优先通过 gateway bootstrap 聚合，不应在页面里直接散连 identity 与 tenant 服务。

## Residual Risks

- 当前 `platform` 模式仍依赖 backend `dev-login`，它是平台 token 接入层，不等于生产级身份治理已完成。
- 本次只把租户开关落到导航和 AI 根入口，其他页面的细粒度 entitlement 仍需后续补齐。