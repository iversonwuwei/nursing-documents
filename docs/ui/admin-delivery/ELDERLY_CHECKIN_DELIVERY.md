# Elderly Checkin Delivery Unit

## Scope

- Entry route: src/app/elderly/checkin/page.tsx
- Affected users: 入住接待、护理主管、入院计划协同用户
- Rollout stage: 第十一批长者工作流子路由治理说明

## User Impact

- 办理入住页当前承接录入、AI 分级建议、人工确认、护理计划与提醒闭环演示。
- 页面现在正式承接来自新增老人页的入口，并通过 URL 参数自动定位新建记录。
- 保持当前 demo 闭环的人工确认出口和计划生成逻辑不变，但把新建入口与审核入口统一到同一条链路。

## Data Source

- Route type: client workflow page with shared external store
- Primary source: admission-workflow shared mock store、统一校验函数和 URL 查询参数
- Downstream dependencies: addAdmissionApplication, confirmAdmissionPlan, markAdmissionAsAdmitted and reminder or task derivations

## UI States

- Loading state: 当前闭环以本地 store 同步更新，后续接真实入住流时需补阶段操作反馈。
- Empty state: 若入住记录为空，应显式提示暂无入住样本并保留新建入口。
- Error state: 表单校验失败、人工调整说明缺失、selected 参数失效或闭环阶段口径不一致时应显式暴露。
- Mobile impact: 多模块长页面、统计卡和右侧执行回执区域需要验证窄屏滚动和锚点可达性。

## Health Signals

- Healthy signal: 新建页录入、AI 建议、人工确认和闭环进度在同一 shared mock store 上下文下保持一致。
- Failure signal: 入住状态、任务提醒和执行回执口径分叉，或 query 中的 selected 对象与右侧详情错位。
- Verification proxy: lint 通过；行为改动时加 build 与入住闭环人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证从 `/elderly/new` 跳入本页、自动选中新建对象、人工确认计划和标记已入住后，列表、统计和回执同步更新

## Rollback

- Revert this delivery note and any future elderly checkin route changes together.
- If regressions appear, fallback is the current shared mock workflow loop.