# Incidents Delivery Unit

## Scope

- Entry route: src/app/incidents/page.tsx
- Affected users: 质控、运营、值班管理与事故复盘用户
- Rollout stage: 第十五批运营新建闭环治理说明

## User Impact

- 事故列表页现在承担事故搜索、级别筛选、待分派提示、AI 复盘摘要和 AI 跟进行动建议。
- 新增报告提交后会回流到本页，先进入待分派闭环，再由值班主管启动处置。
- 列表统计、详情入口和待分派提示卡现在围绕同一份共享 workflow 数据集保持一致。

## Data Source

- Route type: client page with local search/filter state and shared workflow subscription
- Primary data: shared operations workflow mock store plus admin AI helpers
- Upstream link: incidents new route
- Downstream links: incident detail pages and AI assistant contextual links

## UI States

- Loading state: 当前为本地 workflow store，后续接真实事故流需补搜索、筛选和待分派推进中的可见反馈。
- Empty state: 搜索或筛选无结果时保持 EmptyState 搜索空态。
- Error state: 事故列表、待分派提示卡、状态统计和 AI 复盘建议不一致时应局部暴露，不能整页静默。
- Mobile impact: 列表卡片、待分派提示卡和 AI 区块并存，需确认窄屏滚动顺序与点击区域。

## Health Signals

- Healthy signal: 新增事故能回流列表，待分派状态、处置动作、AI 摘要和详情链接围绕同一事故数据集保持一致。
- Failure signal: 新增事故未显示待分派，或开始处置后列表状态未更新。
- Verification proxy: docs build、lint、build 通过；人工验证新建事件 -> 待分派 -> 开始处置 -> 详情查看闭环。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新建事故回流、待分派提示卡、开始处置动作、搜索、级别筛选、AI 摘要和详情跳转

## Rollback

- Revert this delivery note together with incidents new route and shared operations workflow 接入。
- If later changes regress, fallback is the previous static incidents list and local search/filter implementation.
