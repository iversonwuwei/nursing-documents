# Notifications Delivery Unit

## Scope

- Entry route: src/app/notifications/page.tsx
- Affected users: 值班管理、入住协同、护理执行协调与消息中心用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 提醒中心承担共享 workflow store 派生提醒、筛选、备注保存、升级处理和已读处理的统一入口。
- 当前交付单元先固定说明与验证门禁，不修改提醒筛选、备注保存和升级处理行为。
- 保持该页仍停留在本地共享 store 范围，不引入真实消息系统或发送通道契约。

## Data Source

- Route type: client page with local filters plus useSyncExternalStore subscription
- Primary sources: admission-workflow store, derived reminder items, and local reminder mutation helpers
- Downstream dependencies: reminder audit note persistence and local status transitions

## UI States

- Loading state: 当前依赖本地共享 store，同步渲染；若未来接真实提醒系统需补列表和保存反馈。
- Empty state: 搜索或状态筛选无提醒时应保持列表级空态。
- Error state: 提醒统计、提醒状态和备注/升级保存结果不一致时需局部暴露。
- Mobile impact: 筛选条、提醒卡片、备注输入和状态动作并存，后续改动需验证窄屏编辑与 CTA 可达性。

## Health Signals

- Healthy signal: 提醒统计、筛选结果、状态推进和备注保存围绕同一 reminder 数据集保持一致。
- Failure signal: 状态推进与统计错位，或本地消息原型越过真实消息系统边界做出不可回退承诺。
- Verification proxy: lint 通过；行为改动时加 build 与提醒中心人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、状态筛选、备注保存、升级处理和已读/处理链路

## Rollback

- Revert this delivery note and any future notifications route changes together.
- If regressions appear, fallback is the previous local reminder feed, filter logic, and status mutation behavior.
