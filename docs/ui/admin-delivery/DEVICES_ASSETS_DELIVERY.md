# Devices Assets Delivery Unit

## Scope

- Entry route: src/app/devices/assets/page.tsx
- Affected users: 设备台账管理、资产运维、后勤协同用户
- Rollout stage: 第七批设备子路由治理说明

## User Impact

- 设备资产页不再用 StandardModulePage 静态演示数据，而是读取真实设备台账，按类别聚合，展示在用/维保/退役/总库存等资产口径。
- 管理员可以直接看到 Operations Service 中的实际设备台账（总数、各类别、在用/维保/退役分布），并从列表点入任一设备到 `/equipment/[id]` 详情。
- 当前交付只涉及只读聚合视图，不涉及台账新建、报废等写操作（仍走 `/equipment` 主页）。

## Data Source

- Route type: live equipment asset aggregation page
- Primary source: `fetchAdminEquipment({ pageSize: 500 })` in `src/lib/services/admin-operations-services.ts` -> `/api/admin-operations/equipment` -> Admin BFF `/api/admin/equipment` -> Operations Service `/api/operations/equipment`
- Derived aggregates: 前端基于返回列表按 `category`、`status`、`lifecycleStatus` 统计分组，不引入新后端聚合端点。
- Removed source: `src/lib/data/standard-pages.tsx` 里的 `devicesAssetsPage` 静态配置不再被该路由消费。

## UI States

- Loading state: 首次加载或手动刷新时显式展示“加载中”占位，不保留旧数据。
- Empty state: 后端返回空台账时展示空态，并提供回到 `/equipment` 主页新建的入口。
- Error state: BFF 或 Operations 不可达时显示错误文案与重试按钮，不回退到 mock。
- Mobile impact: 类别分组卡片在窄屏下单列堆叠，列表使用已有 `table-wrap` 保持可滚动。

## Health Signals

- Healthy signal: 资产总数、类别分布、状态分布与 `/equipment` 主列表口径一致（同一分页下）。
- Failure signal: 页面出现静态示例名称（`DEV-*` demo）、或类别分布和 `/equipment` 主页不一致。
- Verification proxy: `npm run lint` + `npm run build` + `docs:build`。

## Verification

- Minimum gate: `npm run lint`
- Required gate for behavior change: `npm run lint` + `npm run build`
- Manual path: 登录 admin -> `/devices/assets`，默认看到真实类别聚合；停 Operations 服务或 BFF，应看到错误态与重试。

## Rollback

- Revert this delivery note 与 `src/app/devices/assets/page.tsx` 前端 commit 即可回到旧的 StandardModulePage 静态视图。
- 由于复用已有 `/api/admin/equipment` 聚合，无后端/BFF 端点需同步回滚。
