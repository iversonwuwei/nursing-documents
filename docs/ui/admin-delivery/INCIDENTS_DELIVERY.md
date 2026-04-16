# Incidents Delivery Unit

## Scope

- Entry route: src/app/incidents/page.tsx
- Affected users: 质控、运营、值班管理与事故复盘用户
- Rollout stage: 第十六批事故工作台主区收口与帮助页后置

## User Impact

- 事故列表页现在承担事故搜索、级别筛选、待分派提示、优先处置队列和详情入口。
- AI 复盘摘要、趋势叙事和长说明后置到右侧信息轨与帮助页，首屏主区不再堆叠解释型区块。
- 新增报告提交后会回流到本页，先进入待分派闭环，再由值班主管启动处置。
- 列表统计、优先队列、详情入口和待分派提示卡现在围绕同一份共享 workflow 数据集保持一致。

## Data Source

- Route type: client page with local search/filter state and shared workflow subscription
- Primary data: shared operations workflow mock store plus admin AI helpers
- Upstream link: incidents new route
- Downstream links: incident detail pages, help route and AI assistant contextual links

## UI States

- Loading state: 当前为本地 workflow store，后续接真实事故流需补搜索、筛选和待分派推进中的可见反馈。
- Empty state: 搜索或筛选无结果时保持 EmptyState 搜索空态。
- Error state: 事故列表、待分派提示卡、状态统计、优先队列和右轨 AI 摘要不一致时应局部暴露，不能整页静默。
- Mobile impact: 列表卡片、待分派提示卡、右轨 AI 区块和帮助入口并存，需确认窄屏滚动顺序与点击区域。

## Health Signals

- Healthy signal: 新增事故能回流列表，待分派状态、处置动作、优先队列、右轨 AI 摘要、帮助页入口和详情链接围绕同一事故数据集保持一致。
- Failure signal: 新增事故未显示待分派，开始处置后列表状态未更新，或右轨 AI 解释与主工作区事故状态脱节。
- Verification proxy: docs build、lint、build 通过；人工验证新建事件 -> 待分派 -> 开始处置 -> 详情查看 -> 帮助页回跳闭环。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新建事故回流、待分派提示卡、优先队列、开始处置动作、搜索、级别筛选、右轨 AI 摘要、帮助页跳转和详情跳转

## Rollback

- Revert this delivery note together with incidents list route, help route and shared operations workflow 接入。
- If later changes regress, fallback is the previous static incidents list and local search/filter implementation.
