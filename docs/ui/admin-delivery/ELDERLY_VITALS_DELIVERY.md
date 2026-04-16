# Elderly Vitals Delivery Unit

## Scope

- Entry route: src/app/elderly/vitals/page.tsx
- Affected users: 生命体征录入、巡诊护士、护理主管用户
- Rollout stage: 体征记录页主区收口与帮助后置批次

## User Impact

- 指标更新页当前承接生命体征记录汇总、搜索过滤和批量录入入口。
- 主区优先保留 KPI、筛选和体征表格，趋势解释与页面说明后置到信息轨。
- 保持体征列表与长者详情跳转关系不变。

## Data Source

- Route type: client vitals list page with local search state
- Primary source: in-file RECORDS vital mock data
- Downstream links: elderly detail route links derived from vital record ids

## UI States

- Loading state: 当前搜索过滤为本地即时响应；后续接真实录入批处理时需补上传或保存反馈。
- Empty state: 搜索后无匹配记录时应显式提示无体征记录，而不是展示空表格。
- Error state: 顶部 KPI、趋势指示与表格数据不一致时需显式暴露。
- Mobile impact: KPI 栅格、筛选栏和长表格在窄屏下需要验证横向滚动和可读性。
- Help state: 通过后置信息轨查看趋势判读边界和帮助入口，不再把解释型文案堆回表格主区。

## Health Signals

- Healthy signal: 体征记录页稳定展示当日录入情况，趋势指示、右轨摘要和表格数据口径一致。
- Failure signal: 搜索、趋势图标和明细记录错位，或详情跳转对象错误。
- Verification proxy: lint 通过；行为改动时加 build 与体征录入流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索过滤、体征明细表和详情跳转工作正常

## Rollback

- Revert this delivery note and any future elderly vitals route changes together.
- If regressions appear, fallback is the current local vitals list composition.
