# Settings Audit Logs Delivery Unit

## Scope

- Entry route: src/app/settings/audit-logs/page.tsx
- Affected users: 系统管理员、内容治理与运营配置协同用户
- Rollout stage: 第十九批物资、房间与员工页面主区收口

## User Impact

- 审计日志页承担内容管理变更记录检索、展开查看和回溯核对的对象入口。
- 主工作区优先保留筛选、日志表格和展开详情，把审计口径说明、风险边界和帮助入口后置到信息轨。
- 当前展开中的审计记录摘要移到信息轨，首屏主区不再混排解释性说明。
- 保持日志页只读，不在该页直接执行配置回滚或资源修复动作。

## Data Source

- Route type: client page with async fetch, local filter state, pagination, and row expansion state
- Primary source: content-management-workflow audit log query
- Downstream link: settings root route and dedicated audit logs help route

## UI States

- Loading state: 当前异步请求只在局部刷新，后续若接真实服务需补显式加载反馈。
- Empty state: 搜索或资源类型筛选无结果时保持搜索空态。
- Error state: 查询失败时回退空列表，后续真实服务接入时需补可观测错误提示。
- Mobile impact: 日志表格与展开详情在窄屏下需验证横向滚动与帮助入口可达性。

## Health Signals

- Healthy signal: 时间、操作人、资源类型、快照详情围绕同一审计记录集保持一致。
- Failure signal: 查询条件与分页结果错位，或展开详情展示了错误快照。
- Verification proxy: lint 通过；行为改动时加 build 与审计日志人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证关键词筛选、资源类型筛选、分页、展开详情摘要、帮助入口和返回设置入口

## Rollback

- Revert this delivery note and any future settings audit logs route changes together.
- If regressions appear, fallback is the previous table-only audit log layout and expansion behavior.
