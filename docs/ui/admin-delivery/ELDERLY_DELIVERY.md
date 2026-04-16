# Elderly Route Delivery Unit

## Scope

- Entry route: src/app/elderly/page.tsx
- Affected users: 前台接待、护理主管、运营与档案管理用户
- Rollout stage: 老人台账 Fluent 收敛与主入口强化

## User Impact

- 老人列表页继续承担搜索、筛选、分页和进入详情的主入口职责，但首屏进一步收敛为“台账总览 + 关键治理闭环 + 列表处理区”。
- 新增老人与资料导入继续进入统一治理链路，但说明性闭环块需要采用 Fluent 风格的轻量治理卡，而不是旧式统计块。
- 人脸录入状态与快捷动作继续保留在主列表，避免前台在详情页和人脸页之间反复跳转查找对象。
- 页面上下文说明收敛到后置区域或帮助页，不再与主筛选和列表并排竞争首屏注意力。

## Data Source

- Route type: client page with local filter state + shared workflow subscriptions
- Primary data: static elderlyList plus admission-workflow shared store merged through elderly-registry helper, and face-enrollment workflow shared store for人脸状态
- Downstream links: elderly detail pages, elderly new page, elderly import page, checkin workflow entry, and face enrollment workflow entry
- Visual scope: Microsoft Fluent 2 inspired page-level restyle only; no contract or store changes introduced

## UI States

- Loading state: 当前为本地数据加共享 store，后续接接口时需要补首屏加载与分页切换反馈。
- Empty state: 搜索或筛选无结果时维持 EmptyState 搜索空态。
- Error state: 若 shared store 与静态台账口径不一致，应优先暴露列表映射异常，不能静默吞掉。
- Mobile impact: KPI、治理闭环卡和筛选区继续保持纵向阅读顺序；表格在较窄宽度下仍需确认横向滚动和操作列可达性。

## Health Signals

- Healthy signal: 首屏先完成“判断当前台账压力 -> 进入审核、导入或详情处理”的闭环；搜索、护理等级筛选、状态筛选和分页保持稳定组合。
- Failure signal: 页面重新堆入训练性说明卡，或 shared store 新建/导入记录未出现在列表、人脸状态与人脸录入页不一致。
- Verification proxy: lint 通过；行为变更时加 build 与手工列表流回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证新增老人入口、资料导入入口、入住审核入口、搜索筛选分页、详情进入、人脸状态展示、人脸快捷动作，以及新建或导入对象回流列表

## Rollback

- Revert this route delivery note and any future elderly list route changes together.
- If later list behavior regresses, fallback is the previous static台账列表实现，并移除对 shared workflow store 的订阅。
