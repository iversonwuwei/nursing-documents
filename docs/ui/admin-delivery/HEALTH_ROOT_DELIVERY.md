# Health Root Delivery Unit

## Scope

- Entry route: `src/app/health/page.tsx`
- Affected users: 监控值班、护理主管
- Rollout stage: 与 `/health-monitoring` 保持同构，不承担独立数据源

## Change Summary

- `/health` 路由继续作为 `/health-monitoring` 的 re-export 别名：`export { default } from '../health-monitoring/page'`。
- 数据源、UI 行为、健康信号、验证口径均与 Health Monitoring Delivery Unit 一致，不额外维护。
- 若将来需要独立首屏，应先在 Health Monitoring 本身完成拆分，再替换本路由的导入。

## Data Source

- Same as `HEALTH_MONITORING_DELIVERY.md`：`fetchAdminVitals({ take: 500 })` → `/api/admin-vitals/vitals` → Admin BFF → Health Service。

## Verification

- 与 `HEALTH_MONITORING_DELIVERY.md` 一致；本路由不需要单独的构建或手测步骤。

## Rollback

- 回退方式：恢复原 re-export 行为即可，本 Unit 不对该文件做结构变更。
