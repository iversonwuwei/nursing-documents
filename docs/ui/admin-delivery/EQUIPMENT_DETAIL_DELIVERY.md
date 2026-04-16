# Equipment Detail Delivery Unit

## Scope

- Entry route: src/app/equipment/[id]/page.tsx
- Affected users: 设备工程、护理协同、设备维保跟进用户
- Rollout stage: 第十七批分析与详情页主区收口

## User Impact

- 设备详情页当前承担单设备状态、实时指标、历史波动和维保信息的汇总查看。
- 主区应只保留单设备状态、实时指标、历史数据和对象事实，AI 解释、维保叙述和页面说明迁移到后置上下文与帮助页。
- 保持设备详情对 AI 运营中心的跳转语义和单设备追踪入口不变。

## Data Source

- Route type: client detail page with route param fallback
- Primary source: in-file DEVICE_DATA mock detail payload
- Downstream dependencies: getEquipmentDetailAiInsight, getEquipmentMaintenanceNarratives, buildAiAssistantHref

## UI States

- Loading state: 当前为本地静态详情，无显式加载态；后续接真实详情接口时需补参数切换反馈。
- Empty state: 未命中设备 id 时回退到默认设备样例；后续接真实数据时应改为显式未找到状态。
- Error state: AI 解释、维保摘要或历史指标口径不一致时应显式暴露，而不是静默展示默认值。
- Mobile impact: 顶部操作区、实时指标卡、历史表格、对象事实区和帮助入口在窄屏下需要验证可读性。

## Health Signals

- Healthy signal: 设备详情页稳定展示单设备状态、历史数据和 AI 跳转上下文，且说明型内容后置后不影响对象核对流程。
- Failure signal: 路由参数失效、详情回退被误当成真实命中，或 AI 上下文跳转到错误设备。
- Verification proxy: lint 通过；行为改动时加 build 与单设备详情人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证 `/equipment/EQ001` 进入后可看到实时指标、历史表格、后置 AI 卡片、对象事实和帮助入口

## Rollback

- Revert this delivery note and any future equipment detail route changes together.
- If regressions appear, fallback is the current single-device mock detail view with fixed AI linkage.