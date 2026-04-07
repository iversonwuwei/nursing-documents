# Nursing Check-in Delivery Unit

## Scope

- Entry route: src/app/nursing/[module]/page.tsx with `/nursing/checkin`
- Affected users: 值班主管、护理组长、机构运营
- Rollout stage: 第 21 批打卡管理工作流补齐

## User Impact

- `/nursing/checkin` 不再重定向到任务页，而是承接独立的打卡管理台。
- 管理端可统一查看待执行、服务中、异常待复核、待主管确认和已确认的打卡记录。
- 主管确认动作只确认闭环质量，不直接改写排班或护理计划。

## Data Source

- Route type: client page with `useSyncExternalStore`
- Primary source: nursing-service-workflow 的打卡记录读模型与观察指标
- Downstream links: `/staff/tasks`、`/staff/schedule`、AI assistant context links

## UI States

- Loading state: workflow store 刷新时显示同步状态。
- Empty state: 当前筛选下无打卡记录时显示列表级空态。
- Error state: workflow 同步失败或主管确认失败时显示卡片级错误。
- Mobile impact: 记录卡片和筛选条需要在窄屏下保持可读，不依赖宽表格。

## Health Signals

- Healthy signal: 统计卡、筛选结果、异常记录和主管确认动作围绕同一批打卡记录保持一致。
- Failure signal: 任务状态已变化但打卡记录未同步，或主管确认后状态未更新。

## Verification

- `npm run lint`
- `npm run build`
- Manual path: 任务执行状态变化后，验证 `/nursing/checkin` 的记录、异常和主管确认动作

## Rollback

- Revert `/nursing/checkin` route implementation and workflow record extensions.
- Fallback is the previous redirect to `/staff/tasks`.