# Elderly Route Delivery Unit

## Scope

- Entry route: src/app/elderly/page.tsx
- Affected users: 前台接待、护理主管、运营与档案管理用户
- Rollout stage: 第一批高频路由治理说明

## User Impact

- 老人列表页承担搜索、筛选、分页和进入详情的主入口职责。
- 当前交付单元先明确边界和验证门禁，不调整列表功能。
- 保持现有批量导入、新增老人、列表点击进详情的路径不变。

## Data Source

- Route type: client page with local filter state
- Primary data: lib/data elderlyList
- Downstream links: elderly detail pages and admission-related actions

## UI States

- Loading state: 当前为本地数据，后续接接口时需要补首屏加载与分页切换反馈。
- Empty state: 搜索或筛选无结果时维持 EmptyState 搜索空态。
- Error state: 后续若接真实数据，列表和分页错误应局部暴露，不能整页静默。
- Mobile impact: 表格在较窄宽度下需要确认横向滚动和操作列可达性。

## Health Signals

- Healthy signal: 搜索、护理等级筛选、状态筛选和分页能稳定组合，详情跳转不丢失上下文。
- Failure signal: 列表数量、分页和筛选不一致，或空态/详情入口失效。
- Verification proxy: lint 通过；行为变更时加 build 与手工列表流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、筛选、分页、空态、详情进入五个基本路径

## Rollback

- Revert this route delivery note and any future elderly list route changes together.
- If later list behavior regresses, fallback is the previous table and filter implementation without touching detail routes.