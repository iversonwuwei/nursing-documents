# Activities Delivery Unit

## Scope

- Entry route: src/app/activities/page.tsx
- Affected users: 活动运营、前台协同、护理主管
- Rollout stage: 第三批高频路由治理说明

## User Impact

- 活动管理页承担活动总览、搜索、状态浏览和进入活动详情的主入口。
- 当前交付单元先固定说明与验证门禁，不改现有列表与搜索行为。
- 保持今日统计、新建活动入口和活动卡片跳转不变。

## Data Source

- Route type: client page with local search state
- Primary data: local activities mocks
- Downstream links: activity detail routes

## UI States

- Loading state: 当前为本地数据，后续接活动排期接口时需补列表加载与搜索反馈。
- Empty state: 搜索无结果时保持 EmptyState 搜索空态。
- Error state: 列表、统计和详情入口若不一致，需要局部可见而非静默。
- Mobile impact: 活动卡片信息密度高，后续变更需验证窄屏下日期、地点和人数信息不挤压关键 CTA。

## Health Signals

- Healthy signal: 搜索、统计和活动列表保持一致，详情入口稳定可达。
- Failure signal: 搜索结果与统计口径不一致，或详情入口错链。
- Verification proxy: lint 通过；行为改动时加 build 与活动列表流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、空态、今日统计和活动详情跳转

## Rollback

- Revert this delivery note and any future activities route changes together.
- If later regressions appear, fallback is the previous activities list and search implementation.