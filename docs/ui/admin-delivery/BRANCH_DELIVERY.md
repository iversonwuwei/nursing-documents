# Branch Delivery Unit

## Scope

- Entry route: src/app/branch/page.tsx
- Affected users: 区域运营、院务管理、多分院统筹用户
- Rollout stage: 分院概览 Fluent 收敛与旧页替换

## User Impact

- 分院管理页继续作为多院区运营概览入口，但页面结构从旧式硬编码卡片收敛为 Fluent 风格的“总览 + 分院优先队列 + 多院区入口”。
- 首屏先帮助区域运营判断哪家分院需要先看，而不是只罗列一串旧式卡片。
- 当前仍不引入新的写路径或编辑流，重点是视觉和信息层级收敛。

## Data Source

- Route type: client branch overview page
- Primary source: in-file BRANCHES mock data
- Downstream dependency: shared PageHeader and StatCard layout
- Visual scope: page-level Fluent 2 restyle using shared cards and overview patterns

## UI States

- Loading state: 当前为本地静态分院数据；后续接真实组织服务时需补总览加载反馈。
- Empty state: 分院列表为空时应显式提示暂无分院，而不是保留空白列表区。
- Error state: 分院统计与卡片明细口径不一致时应显式暴露。
- Mobile impact: 总览区、优先队列和分院入口卡在窄屏下继续保持纵向堆叠，不依赖横向指标条才可读。

## Health Signals

- Healthy signal: 用户进入页面后能先判断哪家分院优先级最高，再进入分院详情；总数、床位统计和分院入口卡数据保持一致。
- Failure signal: 页面继续停留在旧式扁平列表，或头部统计与分院入口卡数据冲突。
- Verification proxy: lint 通过；行为改动时加 build 与分院页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证分院页可查看汇总 KPI 与分院列表，且统计与卡片数据一致

## Rollback

- Revert this delivery note and any future branch route changes together.
- If regressions appear, fallback is the current local branch overview list.