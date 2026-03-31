# Equipment Detail Delivery Unit

## Scope

- Entry route: src/app/equipment/[id]/page.tsx
- Affected users: 设备工程、护理协同、设备维保跟进用户
- Rollout stage: 第九批设备源路由治理说明

## User Impact

- 设备详情页当前承担单设备状态、实时指标、历史波动和维保信息的汇总查看。
- 当前交付单元先固定页面职责、验证门禁和回滚路径，不修改详情视图行为。
- 保持设备详情对 AI 运营中心的跳转语义和单设备追踪入口不变。

## Data Source

- Route type: client detail page with route param fallback
- Primary source: in-file DEVICE_DATA mock detail payload
- Downstream dependencies: getEquipmentDetailAiInsight, getEquipmentMaintenanceNarratives, buildAiAssistantHref

## UI States

- Loading state: 当前为本地静态详情，无显式加载态；后续接真实详情接口时需补参数切换反馈。
- Empty state: 未命中设备 id 时回退到默认设备样例；后续接真实数据时应改为显式未找到状态。
- Error state: AI 解释、维保摘要或历史指标口径不一致时应显式暴露，而不是静默展示默认值。
- Mobile impact: 顶部操作区、实时指标四列卡片和历史表格在窄屏下需要验证可读性。

## Health Signals

- Healthy signal: 设备详情页稳定展示单设备状态、历史数据和 AI 跳转上下文，且 id 切换不会丢失详情语义。
- Failure signal: 路由参数失效、详情回退被误当成真实命中，或 AI 上下文跳转到错误设备。
- Verification proxy: lint 通过；行为改动时加 build 与单设备详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/equipment/EQ001` 进入后可看到实时指标、历史表格、维保摘要和 AI 运营中心入口

## Rollback

- Revert this delivery note and any future equipment detail route changes together.
- If regressions appear, fallback is the current single-device mock detail view with fixed AI linkage.