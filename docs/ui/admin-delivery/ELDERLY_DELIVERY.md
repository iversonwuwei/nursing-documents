# Elderly Route Delivery Unit

## Scope

- Entry route: src/app/elderly/page.tsx
- Affected users: 前台接待、护理主管、运营与档案管理用户
- Rollout stage: 第一批高频路由治理说明

## User Impact

- 老人列表页仍承担搜索、筛选、分页和进入详情的主入口职责。
- 列表现在同时展示静态台账与新建闭环中的对象，并在顶部暴露入住审核入口。
- 新增老人与资料导入不再是孤立按钮，而是统一进入“录入或导入 -> 审核 -> 入住 -> 台账”的治理链路。
- 列表需要直接回挂人脸录入状态，并提供行级快捷动作，避免前台在详情页和人脸页之间反复跳转查找对象。

## Data Source

- Route type: client page with local filter state + shared workflow subscriptions
- Primary data: static elderlyList plus admission-workflow shared store merged through elderly-registry helper, and face-enrollment workflow shared store for人脸状态
- Downstream links: elderly detail pages, elderly new page, elderly import page, checkin workflow entry, and face enrollment workflow entry

## UI States

- Loading state: 当前为本地数据加共享 store，后续接接口时需要补首屏加载与分页切换反馈。
- Empty state: 搜索或筛选无结果时维持 EmptyState 搜索空态。
- Error state: 若 shared store 与静态台账口径不一致，应优先暴露列表映射异常，不能静默吞掉。
- Mobile impact: 表格在较窄宽度下需要确认横向滚动和操作列可达性。

## Health Signals

- Healthy signal: 搜索、护理等级筛选、状态筛选和分页能稳定组合，新建与导入对象都能进入入住审核并回流列表，人脸状态与人脸页口径一致。
- Failure signal: shared store 新建或导入记录未出现在列表、入口跳转错误、详情映射失效，或人脸状态与人脸录入页不一致。
- Verification proxy: lint 通过；行为变更时加 build 与手工列表流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新增老人入口、资料导入入口、入住审核入口、搜索筛选分页、详情进入、人脸状态展示、人脸快捷动作，以及新建或导入对象回流列表

## Rollback

- Revert this route delivery note and any future elderly list route changes together.
- If later list behavior regresses, fallback is the previous static台账列表实现，并移除对 shared workflow store 的订阅。
