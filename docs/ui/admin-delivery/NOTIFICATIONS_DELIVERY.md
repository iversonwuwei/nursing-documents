# Notifications Delivery Unit

## Scope

- Entry route: src/app/notifications/page.tsx
- Affected users: 值班管理、入住协同、护理执行协调与消息中心用户
- Rollout stage: 第八批详情与根路由治理说明

## User Impact

- 通知中心只展示真实 Notification Service 摘要、发送队列和失败待处置消息，不再回退共享 workflow store 提醒闭环。
- 当前交付单元把页面收敛为 live-only 读视图；未接通的备注保存、已读、升级处理不再由前端本地 store 伪造。
- 当真实通知服务不可用时，页面保留错误状态和空态，不再继续展示 demo 通知或本地提醒队列。

## Data Source

- Route type: client page with live Notification Service snapshot only
- Primary sources: `/api/content/notifications/summary` and `/api/content/notifications/queue`
- Downstream dependencies: Notification Service queue status, category breakdown, and live empty/error states

## UI States

- Loading state: 首屏读取真实通知摘要与消息队列时显示同步文案。
- Empty state: 真实队列为空时显示 live empty，不再回退本地提醒。
- Error state: Notification Service 不可用时应显式暴露 unavailable 状态，而不是继续渲染 demo 通知。
- Mobile impact: 队列筛选、消息卡片和失败待处置区需验证窄屏下滚动与 CTA 可达性。

## Health Signals

- Healthy signal: 通知摘要、实时队列、失败待处置和通道计数围绕同一真实消息数据集保持一致。
- Failure signal: 页面在真实服务失败时回退 demo 数据，或 live 队列与摘要计数口径不一致。
- Verification proxy: lint 通过；行为改动时加 build 与提醒中心人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 live 摘要、live 队列、失败待处置、搜索/状态筛选与真实空态/错误态

## Rollback

- Revert this delivery note and any future notifications route changes together.
- If regressions appear, rollback should restore the previous notifications route implementation; do not reintroduce local reminder feed as a long-term source of truth.
