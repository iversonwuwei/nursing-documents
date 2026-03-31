# Branch Delivery Unit

## Scope

- Entry route: src/app/branch/page.tsx
- Affected users: 区域运营、院务管理、多分院统筹用户
- Rollout stage: 第十批高频入口治理说明

## User Impact

- 分院管理页当前展示分院总览、床位占用、员工规模和分院级经营信息。
- 当前交付单元先固定分院总览职责和验证门禁，不修改分院卡片展示行为。
- 保持当前分院列表作为多院区运营概览入口，不引入新交互。

## Data Source

- Route type: client branch overview page
- Primary source: in-file BRANCHES mock data
- Downstream dependency: shared PageHeader and StatCard layout

## UI States

- Loading state: 当前为本地静态分院数据；后续接真实组织服务时需补总览加载反馈。
- Empty state: 分院列表为空时应显式提示暂无分院，而不是保留空白列表区。
- Error state: 分院统计与卡片明细口径不一致时应显式暴露。
- Mobile impact: 分院卡片横向指标区在窄屏下需要验证压缩与换行表现。

## Health Signals

- Healthy signal: 分院总数、床位统计和分院卡片明细保持一致，作为多分院概览入口稳定可用。
- Failure signal: 头部统计与列表卡片数据冲突，或分院概览在窄屏下失去可读性。
- Verification proxy: lint 通过；行为改动时加 build 与分院页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证分院页可查看汇总 KPI 与分院列表，且统计与卡片数据一致

## Rollback

- Revert this delivery note and any future branch route changes together.
- If regressions appear, fallback is the current local branch overview list.