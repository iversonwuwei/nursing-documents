# AI Inference Delivery Unit

## Scope

- Entry route: src/app/ai-assistant/inference/page.tsx
- Affected users: AI 治理、运营管理、质控、需要按上下文解释推理结果的协同用户
- Rollout stage: 第四批扩展路由治理说明

## User Impact

- AI 推理详情页承担模型状态、健康解释样本、入住评估记录和治理跳转的统一查看入口。
- 当前交付单元先固定说明与验证门禁，不修改 tracking context、样本列表或治理跳转行为。
- 保持“只读推理结果，不直接改写业务状态”的边界。

## Data Source

- Route type: client page with query-param context and useSyncExternalStore snapshots
- Primary sources: admin AI mock datasets, admission workflow snapshot, AI context helpers
- Downstream link: AI rules route with appended tracking context

## UI States

- Loading state: 当前大部分为本地同步 mock；若后续接真实推理服务，需补模型状态、样本和表格的加载反馈。
- Empty state: tracking context、context cards 或 related logs 为空时，应保持局部空缺而不破坏整体推理视图。
- Error state: context 与对象级样本、日志或推荐记录映射错位时需显式暴露。
- Mobile impact: 上下文卡、模型状态卡和表格信息量大，后续改动需验证窄屏下可读性和 CTA 可达性。

## Health Signals

- Healthy signal: tracking context、模型状态、健康解释样本与规则治理跳转保持一致。
- Failure signal: 来源上下文丢失、对象样本错链，或推理页越过人工确认边界。
- Verification proxy: lint 通过；行为改动时加 build 与 AI 推理上下文人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证带 context 进入、模型状态、样本记录、规则治理跳转和只读边界

## Rollback

- Revert this delivery note and any future ai inference route changes together.
- If regressions appear, fallback is the previous mock inference view and context jump behavior.
