# Equipment Monitor Delivery Unit

## Scope

- Entry route: src/app/equipment/monitor/page.tsx
- Affected users: 设备值班、后勤巡检、护理站联动用户
- Rollout stage: 第九批设备源路由治理说明

## User Impact

- 设备监控页当前提供实时监控总览、告警记录和 AI 巡检建议，是设备运营主监控入口。
- 当前交付单元先固定监控页职责和验证门禁，不修改刷新、告警或监控卡片行为。
- 保持监控页对设备列表和状态路由的联动入口不变。

## Data Source

- Route type: client monitoring dashboard
- Primary sources: in-file MONITOR_POINTS, STATS, ALERT_HISTORY mock data
- Downstream dependencies: getDeviceAiInsights, getDeviceAiOverview

## UI States

- Loading state: 刷新按钮以本地 refreshing 状态反馈刷新过程，后续接实时流时需补真实请求状态。
- Empty state: 当前假定始终有监控点；后续若监控点为空，应显式提示无在线设备或无监控授权。
- Error state: 监控统计、卡片状态和告警列表口径不一致时必须可见，而不是继续渲染成功视图。
- Mobile impact: 监控设备网格、告警记录列表和顶部统计卡片需要验证窄屏滚动与折行表现。

## Health Signals

- Healthy signal: 监控页的总数、在线离线统计、设备卡片和告警记录保持一致，AI 建议与当前告警口径一致。
- Failure signal: 刷新反馈失效、统计卡片与监控卡片不一致，或告警记录无法承接设备状态变化。
- Verification proxy: lint 通过；行为改动时加 build 与监控页人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证监控总览、刷新按钮、实时监控卡片和告警记录区能共同表达当前设备运行状态

## Rollback

- Revert this delivery note and any future equipment monitor route changes together.
- If regressions appear, fallback is the current local监控 mock dashboard behavior.