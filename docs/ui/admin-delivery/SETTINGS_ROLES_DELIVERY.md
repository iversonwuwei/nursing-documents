# Settings Roles Delivery Unit

## Scope

- Entry route: src/app/settings/roles/page.tsx
- Affected users: 系统管理员、权限配置、角色治理用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- 角色权限页不再以 StandardModulePage 静态模板呈现，而是读取 Identity Service 的角色列表，展示系统预设角色、数据范围、核心能力和高危标识。
- 管理员可以直接看到当前后端认定的角色数量、高危角色数量，权限评估不再基于前端演示常量。
- 本次交付只提供角色的只读列表；权限分配、临时授权等写操作不在本交付单元范围内，后端写 API 尚未开放。

## Data Source

- Route type: live role-list page
- Primary source: `fetchAdminRoles()` in `src/lib/services/admin-identity-services.ts` -> Next proxy `/api/admin-identity/roles` -> Admin BFF `/api/admin/roles` -> Identity Service `/api/identity/roles`
- Identity Service 返回预设系统角色列表（SystemAdmin / CareSupervisor / Caregiver / Nurse / Doctor / Reception 等），包含 id、名称、描述、数据范围、核心能力和高危标识。
- Removed source: `settingsRolesPage` 静态配置不再被该路由消费。

## UI States

- Loading state: 初次加载时显式“加载中”，不保留上次列表。
- Empty state: Identity 返回空列表时显示“暂无角色配置”并给出重试按钮。
- Error state: 服务不可达时显示错误文案与重试按钮，不回退到静态演示。
- Mobile impact: 角色表格复用现有 `table-wrap` 包裹，窄屏可滑动。

## Health Signals

- Healthy signal: 列表条数与 `/settings` 首页 KPI 的“系统角色”数相等。
- Failure signal: 列表出现静态示例角色名称（如 “护理主管” 独立于 Identity 返回结果）或与首页 KPI 冲突。
- Verification proxy: `npm run lint` + `npm run build` + `docs:build`。

## Verification

- Minimum gate: `npm run lint`
- Required gate for behavior change: `npm run lint` + `npm run build`
- Manual path: 进入 `/settings/roles`，确认列表为真实 Identity 结果；停 Identity 服务时页面出错态不回退到静态模板。

## Rollback

- Revert this delivery note 与 `src/app/settings/roles/page.tsx` 前端 commit 即可回到旧的 StandardModulePage 静态视图。
- Identity Service 的 `/api/identity/roles`、Admin BFF `/api/admin/roles` 仅新增只读端点，回退不影响现有功能；若需完全回退，单独 revert Identity/BFF commit即可。
