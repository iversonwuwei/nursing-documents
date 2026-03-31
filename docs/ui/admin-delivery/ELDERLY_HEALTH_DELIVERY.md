# Elderly Health Delivery Unit

## Scope

- Entry route: src/app/elderly/health/page.tsx
- Affected users: 护理主管、巡诊护士、用药协同用户
- Rollout stage: 第十一批长者工作流子路由治理说明

## User Impact

- 长者健康档案页当前承接健康记录汇总、AI 巡诊建议、用药提醒和健康指标表。
- 当前交付单元先固定健康档案入口职责和验证门禁，不修改档案、AI 或用药展示行为。
- 保持健康档案页作为长者健康总览入口，不改变详情入口与 AI 建议结构。

## Data Source

- Route type: client archive page with local search state
- Primary sources: HEALTH_RECORDS, MEDICATIONS, admin AI helpers
- Downstream links: elderly detail route links derived from health records

## UI States

- Loading state: 当前搜索和数据展示为本地即时响应；后续接真实档案服务时需补页面级加载反馈。
- Empty state: 搜索后无匹配档案或无用药项时，应显式提示无结果，而不是只保留空表格或空卡片。
- Error state: 健康指标、AI 建议和用药提醒口径不一致时应显式暴露，而不是继续渲染成功态。
- Mobile impact: 五列统计卡、双栏 AI 区和多列表格在窄屏下需要验证横向滚动与阅读顺序。

## Health Signals

- Healthy signal: 健康档案、AI 巡诊建议和用药提醒在同一对象和时间口径下保持一致。
- Failure signal: 搜索结果、统计卡与档案表不一致，或 AI 建议与健康档案对象错位。
- Verification proxy: lint 通过；行为改动时加 build 与健康档案流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、健康指标表、AI 巡诊建议和用药提醒可共同表达当前档案状态

## Rollback

- Revert this delivery note and any future elderly health route changes together.
- If regressions appear, fallback is the current local health archive composition.