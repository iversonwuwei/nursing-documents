# Settings Delivery Unit

## Scope

- Entry route: src/app/settings/page.tsx
- Affected users: 系统管理员、运营配置管理、权限治理协同用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 系统配置首页不再以 StandardModulePage 静态演示数据呈现，而是聚合已真实对接的配置子页入口（静态文本、下拉选项、操作日志、角色权限），以真实计数作为 KPI。
- 管理员进入 `/settings` 就能看到各配置域当前的真实对接状态和总量已接入，不再从开发阶段的例子和水位接口观感。
- 本次交付不引入系统配置的新写操作，所有子页正常行为保持不变。

## Data Source

- Route type: live settings overview page
- Primary sources:
  - `fetchStaticTexts({ pageSize: 1 })` / `fetchStaticTextNamespaces()` -> `/api/content/static-texts*` -> Admin BFF -> Config Service
  - `fetchOptionGroups({ pageSize: 1 })` -> `/api/content/option-groups*` -> Admin BFF -> Config Service
  - `fetchAuditLogs({ pageSize: 1 })` -> `/api/content/audit-logs*` -> Admin BFF -> Config Service
  - `fetchAdminRoles()` -> `/api/admin-identity/roles` -> Admin BFF `/api/admin/roles` -> Identity Service `/api/identity/roles`
- Removed source: `settingsPage` 静态模块配置不再被该路由消费。

## UI States

- Loading state: 初次进入或手动刷新时显式展示“加载中”，KPI 顶栏在数据就绪前不先展示假值。
- Empty state: 某个配置域返回零记录时明确显示 `0 条`，并给出进入对应子页的入口。
- Error state: 任一汇总请求失败时在该配置域卡片上显式标记异常，不会让整页回退到静态 mock。
- Mobile impact: 配置卡片在窄屏下单列堆叠，重点数值和按钮可见。

## Health Signals

- Healthy signal: `/settings` KPI 和每个子页自身总量口径一致；进入子页后看到的列表长度与首页 KPI 一致。
- Failure signal: 出现静态数字（6/18/3 等演示值）或与子页冲突。
- Verification proxy: `npm run lint` + `npm run build` + `docs:build`。

## Verification

- Minimum gate: `npm run lint`
- Required gate for behavior change: `npm run lint` + `npm run build`
- Manual path: 登录 admin -> `/settings`，确认三个配置域 KPI 不再为静态示例；停 Config / Identity 服务时首页仍可加载剩余配置域。

## Rollback

- Revert this delivery note 和 `src/app/settings/page.tsx` 前端 commit 即可回到旧的 StandardModulePage 静态视图。
- 新增的 Identity `/api/identity/roles`、Admin BFF `/api/admin/roles` 及 Next proxy 只提供只读列表，回退后保留无影响；若需一并回退，单独 revert Identity/BFF commit。
