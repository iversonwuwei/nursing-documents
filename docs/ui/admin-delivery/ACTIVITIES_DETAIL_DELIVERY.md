# Activities Detail Delivery Unit

## Scope

- Entry route: src/app/activities/[id]/page.tsx
- Affected users: 活动运营、前台协同、护理主管
- Rollout stage: 第四批扩展路由治理说明

## User Impact

- 活动详情页承担从活动列表进入后的单活动信息核对与编辑入口承接。
- 当前交付单元只固定说明和验证门禁，不修改活动详情 mock 展示行为。
- 保持返回列表、活动 KPI 卡片和详情字段区块不变。

## Data Source

- Route type: client page with dynamic route param
- Primary data: local ACTIVITY_DATA mocks keyed by id
- Upstream link: activities list route

## UI States

- Loading state: 当前为本地同步 mock，后续接远程详情接口时需补拉取反馈与 skeleton。
- Empty state: 当前未知 id 会回退到默认 mock，后续若改为真实接口需显式区分 not found 与空态。
- Error state: 参数映射错误或详情字段缺失时需局部暴露，不应静默回退掩盖问题。
- Mobile impact: 详情页头、统计卡和属性网格在窄屏下仍需保持返回与编辑 CTA 可见。

## Health Signals

- Healthy signal: 从活动列表进入后标题、统计卡、详情字段和编辑入口保持一致。
- Failure signal: 动态 id 与详情内容错位，或回退策略掩盖真实缺失对象。
- Verification proxy: lint 通过；行为改动时加 build 与活动详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证从活动列表进入详情、返回列表、统计卡和详情字段展示

## Rollback

- Revert this delivery note and any future activities detail route changes together.
- If regressions appear, fallback is the previous local detail mock and list-to-detail link behavior.
