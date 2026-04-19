# Incidents Delivery Unit

## Scope

- Entry route: src/app/incidents/page.tsx
- Affected users: 质控、运营、值班管理与事故复盘用户
- Rollout stage: incidents family live read-write 收口，切换到 Admin Web -> Next proxy -> Admin BFF -> Operations Service

## User Impact

- 事故列表页现在承担事故搜索、级别筛选、待分派提示、优先处置队列和详情入口。
- AI 复盘摘要、趋势叙事和长说明后置到右侧信息轨与帮助页，首屏主区不再堆叠解释型区块。
- 新增报告提交后会回流到本页，先进入待分派闭环，再由值班主管启动处置。
- 列表统计、优先队列、详情入口和待分派提示卡现在围绕同一份共享 workflow 数据集保持一致。

## Data Source

- Route type: client page with local search/filter state and live API read model
- Primary data: Admin incidents list and status action APIs；AI 摘要改为后端 AI incident-analysis
- Upstream link: incidents new route
- Downstream links: incident detail pages, help route and AI assistant contextual links

## UI States

- Loading state: 列表和 AI 摘要首屏请求期间需保持显式加载反馈，不默认显示本地事故样例。
- Empty state: 搜索或筛选无结果时保持 EmptyState 搜索空态；live 返回空列表时保持真实空态。
- Error state: 事故列表、待分派提示卡、状态统计、优先队列和右轨 AI 摘要不一致时应局部暴露，不能整页静默，也不回退本地 workflow。
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

- Revert this delivery note together with incidents list route、Next proxy、Admin BFF、incident-analysis 接入和 operations incidents endpoints。
- If later changes regress, fallback is the previous local operations-workflow and mock AI implementation.
