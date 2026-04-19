# Staff Delivery Unit

## Scope

- Entry route: src/app/staff/page.tsx
- Affected users: 人力运营、护理主管、值班管理与排班协同用户
- Rollout stage: 第五批高频运营路由治理说明

## User Impact

- 员工列表页承担员工搜索、部门筛选、状态浏览与进入详情/排班的主入口职责。
- 当前交付单元把员工列表、新增员工、确认入职和员工详情切到真实 staff API，并要求新建时写入真实机构归属，不再从前端 resource workflow 读取生产数据。
- 保持详情和排班入口仍由人工选择，不把 AI 摘要变成员工绩效判断。

## Data Source

- Route type: client page with local search, filter, and pagination state
- Primary sources: `GET /api/staff`、`GET /api/staff/{staffId}`、`POST /api/staff`、`POST /api/staff/{staffId}/activate`，以及机构选择所需的 `GET /api/organizations`
- Downstream links: staff detail, schedule, and AI assistant context links

## UI States

- Loading state: 首屏显示 staff live 请求中的加载提示；新建与确认入职按钮展示提交中。
- Empty state: 搜索或部门筛选无结果时应保持搜索空态。
- Error state: staff API 不可用、列表与详情口径不一致时需局部暴露。
- Mobile impact: 表格列较多，后续改动需验证窄屏下搜索、筛选和详情 CTA 可达性。

## Health Signals

- Healthy signal: KPI、筛选结果、员工列表、机构归属、详情/排班入口围绕同一员工数据集保持一致。
- Failure signal: 列表与统计口径错位、员工误绑机构，或 AI 摘要越过“结构建议而非绩效判断”的边界。
- Verification proxy: lint 通过；行为改动时加 build 与员工列表人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、部门筛选、空态、详情入口、排班入口和 AI 链路

## Rollback

- Revert this delivery note and any future staff route changes together.
- If regressions appear, rollback path is to restore the previous frontend-only staff list, creation, onboarding, and detail composition.
