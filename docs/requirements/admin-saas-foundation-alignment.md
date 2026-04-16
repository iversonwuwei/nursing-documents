# Admin SaaS 基线对齐

## Scope

- scope: 为 nursing-admin-v2 落地第一批真正面向 SaaS 的前端基线，包括动态租户上下文、可切换身份源、租户级功能开关三项能力。
- affected users: 超级管理员、机构管理员、租户运营管理员、平台实施与联调人员。
- changed behavior: 登录页可显式选择租户；会话中保留租户、授权来源与租户 entitlements；admin 前端代理优先复用登录时签发的 access token 与 tenant；AI 入口开始遵守租户模块开关。
- dependent systems: `src/app/api/auth/[...nextauth]/route.ts`、登录页、admin BFF 代理、护理 workflow 代理、顶部导航、AI 入口页、Playwright smoke。
- verification: nursing-documents `npm run docs:build`，nursing-admin-v2 `npm run lint`、`npm run build`、聚焦 navbar/scene smoke 与租户能力 smoke。
- rollback: 回退登录页租户输入、session tenant 字段、BFF token 复用逻辑、租户开关 UI 与相关 smoke。

## 目标

- 把前端固定 `tenant-demo` 的实现替换为登录驱动或会话驱动的动态租户上下文。
- 把当前仅本地 credentials 的身份模型升级为“demo / platform 可切换”的身份源抽象，允许前端在平台模式下复用后端 identity service 的 token issuance。
- 把租户级模块与 feature entitlements 收口成统一运行时开关，而不是散落在页面里手工写条件。

## 用户影响

- 登录时可看到当前进入的是哪个租户，切换租户后导航、AI 能力和代理请求都会跟着切换。
- 平台联调模式下，Admin 前端不再为每个代理请求重复临时签 token，而是优先复用登录态里已有的平台 token。
- 未启用 AI 模块的租户不再在主导航里看到 `AI助手`，直接进入 AI 页面时也会看到显式的未启用提示。

## 行为范围

| 能力 | 本次目标 | 不在本次范围 |
| --- | --- | --- |
| 租户上下文 | 登录页选择租户、session 带 tenant、代理层复用 tenant | 真正的域名级 tenant discovery、租户开通流程 |
| 身份源 | `demo` / `platform` 两种模式切换，platform 模式可接 identity dev-login | 完整用户名密码校验、SSO、MFA |
| 功能开关 | 统一读取 tenant modules / features，并至少落地到导航与 AI 入口 | 全量页面的功能矩阵与套餐计费控制面 |

## 验收方向

- 登录后顶部导航与用户区应能显式展示当前租户。
- 平台模式下，admin 代理不应继续强依赖固定 `tenant-demo` 或为每次请求重新假定租户。
- `tenant-private` 这类未启用 AI 模块的租户，在导航和 AI 根入口上都应看到一致的禁用行为。
- 在不改其他业务页面语义的前提下，lint、build 和现有 scene smoke 仍需保持通过。

## 边界说明

- 本次优先完成 admin 前端的 SaaS 接入层，不修改 backend 服务契约本身。
- identity service 当前仍是 dev-login 发行 token，本次把它接入前端认证流，但不把它包装成生产级 IAM 已完成状态。
- 租户开关先以 tenant service 返回的 `EnabledModules` 和 `EnabledFeatures` 为准，后续若接入真实订阅或计费，再演进到更完整的 entitlement 模型。