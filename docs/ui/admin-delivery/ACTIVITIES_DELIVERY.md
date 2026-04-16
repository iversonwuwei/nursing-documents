# Activities Delivery Unit

## Scope

- Entry route: src/app/activities/page.tsx
- Affected users: 活动运营、前台协同、护理主管
- Rollout stage: 第十六批运营工作台主区收口与帮助页后置

## User Impact

- 活动管理页现在承担活动总览、优先队列、搜索、待发布提示、发布动作和进入活动详情的主入口。
- 页面首屏只保留会影响下一步运营动作的内容，推荐处理路径与长说明后置到右侧信息轨和帮助页。
- 新建活动提交后会回流到本页，先显示待发布提示卡，再由运营确认后开放报名。
- 今日统计、活动列表、优先队列和活动详情入口现在围绕同一份共享 workflow 数据集，避免新建数据丢在孤立表单里。

## Data Source

- Route type: client page with local search state and shared workflow subscription
- Primary data: shared operations workflow mock store with localStorage persistence
- Upstream link: activities new route
- Downstream links: activity detail routes and help route

## UI States

- Loading state: 当前为本地同步 workflow store，后续接活动排期接口时需补列表加载与发布反馈。
- Empty state: 搜索无结果时保持 EmptyState 搜索空态。
- Error state: 列表、今日统计、优先队列、待发布提示卡和详情入口若不一致，需要局部暴露而非静默。
- Mobile impact: 活动卡片、待发布提示卡、右轨路径卡和帮助入口并存时，需验证窄屏下日期、地点和 CTA 不会互相挤压。

## Health Signals

- Healthy signal: 新建活动能回流到列表，待发布状态、优先队列、今日统计、帮助页入口和详情入口围绕同一条数据链路保持一致。
- Failure signal: 新建活动未出现在列表，发布动作未同步更新状态与详情，或右轨说明与主工作区状态脱节。
- Verification proxy: docs build、lint、build 通过；人工验证新建活动 -> 列表待发布 -> 发布 -> 详情查看 -> 帮助页回跳闭环。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新建活动回流列表、待发布提示卡、优先队列、发布动作、搜索、空态、帮助页跳转和详情跳转

## Rollback

- Revert this delivery note together with activities list route, help route and shared operations workflow 接入。
- If regressions appear, fallback is the previous static activities list and detail mock implementation.
