# Equipment Status Delivery Unit

## Scope

- Entry route: src/app/equipment/status/page.tsx
- Affected users: 设备巡检、后勤监控、护理协同用户
- Rollout stage: 第九批设备源路由治理说明

## User Impact

- 设备状态页当前提供状态总览、搜索筛选和 AI 巡检动作，是设备状态总板入口。
- 当前交付单元先固定状态板职责和验证门禁，不修改筛选或表格展示行为。
- 保持设备状态总览与设备兼容路由 `/devices/status` 的承接关系不变。

## Data Source

- Route type: client status board page
- Primary source: in-file DEVICES status dataset
- Downstream dependencies: getEquipmentStatusAiInsights, getEquipmentStatusNarratives, buildAiAssistantHref

## UI States

- Loading state: 当前搜索与状态筛选为本地即时过滤；后续接远端状态流时需补请求反馈。
- Empty state: 搜索无结果或筛选后为空时应显式提示无匹配设备，而不是只保留空表格。
- Error state: 顶部统计、筛选结果和表格行数量不一致时需显式暴露状态口径问题。
- Mobile impact: 筛选按钮组、搜索框和状态表格需要验证窄屏下的折行和横向滚动行为。

## Health Signals

- Healthy signal: 顶部状态统计、筛选结果和 AI 解释保持一致，兼容入口与源路由语义一致。
- Failure signal: 统计汇总与筛选表格不一致，或 `/equipment/status` 与 `/devices/status` 状态口径分叉。
- Verification proxy: lint 通过；行为改动时加 build 与状态路由人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证状态总览、搜索筛选、设备表格和 AI 巡检动作在 `/equipment/status` 下工作正常

## Rollback

- Revert this delivery note and any future equipment status route changes together.
- If regressions appear, fallback is the current local状态总板 and devices status compatibility route.