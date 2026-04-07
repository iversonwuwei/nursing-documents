# Admin And Nani Clock-in Workflow

## Scope

- scope: 定义护工端服务打卡与管理端打卡管理的共享状态语义、页面边界和回滚路径。
- boundaries: 仅覆盖 nursing-nani-app 与 nursing-admin-v2 的前端 mock workflow，不扩展真实后端接口。
- dependencies: 任务中心、护理执行、交接班、报警、admin 护理 workflow store。
- failure modes: 护工端未带入打卡摘要、admin 记录与任务状态不一致、异常记录无法进入主管确认。
- verification: `npm run docs:build`，并分别通过 admin 与 nani 的本地代码门禁。
- rollback: 撤回新增 route、模型字段、store 派生逻辑和本文件。

## Workflow Boundaries

- Nani 负责生成服务打卡草稿，最小字段包括任务、对象、房间、打卡方式、到场时间、异常备注。
- 护理执行页负责消费打卡草稿，并继续补执行留证、执行结果和下游交接或报警动作。
- Admin 负责消费统一的打卡记录读模型，提供统计、筛选、异常观察和主管确认动作。
- 当前两端通过各自本地 store 模拟协同，不引入跨仓库直接共享代码。

## Shared State Semantics

### Nani 侧打卡状态

- `待打卡`: 任务已进入本班次，但还未确认到场。
- `已到场`: 已完成到场确认，待进入正式服务执行。
- `服务中`: 已进入护理执行，等待结果提交。
- `异常待复核`: 打卡或执行阶段出现阻断，需要主管或下一班复核。
- `已完成`: 服务执行与打卡摘要都已闭环，可进入主管确认。

### Admin 侧管理状态

- `待执行`: 尚未开始打卡或未到场。
- `服务中`: 已打卡并进入执行，但尚未闭环。
- `异常待复核`: 已有异常说明或整改信号，需要主管介入。
- `待主管确认`: 服务已完成，但尚未形成管理确认。
- `已确认`: 主管已确认记录完整，可留作审计与复盘。

## Data Shape

- `CareClockInDraft`: nani 页面间传递的打卡草稿对象，带任务、对象、打卡方式、位置、时间与异常说明。
- `ServiceClockInRecord`: admin 管理页消费的读模型，带任务、班次、服务人、状态、时间戳、异常原因、主管确认字段。
- `CareExecutionFollowupDraft`: 在现有执行摘要上追加打卡上下文，确保交接班和报警页能看到“何时、以何种方式到场”。

## Healthy Signals

- Nani 从首页或任务页进入打卡后，护理执行页能显示对应打卡摘要。
- Admin 打卡管理页统计、筛选结果和底层记录使用同一批打卡记录。
- 异常待复核记录能被主管确认，不会永远停留在中间态。

## Current Implementation Snapshot

- nani 使用 `CareClockInDraft` 在页面间传递打卡上下文，当前由 `mock_nani_service.dart` 持有内存态。
- admin 使用 `ServiceClockInRecord` 作为管理读模型，当前由 `nursing-service-workflow.ts` 从任务快照派生。
- `/nursing/checkin` 已经从重定向改为真实管理页面；`care_checkin` 已经成为护理执行前置入口。
- 当前实现刻意不做跨端实时同步，先保证单端流程、状态定义和验证门禁稳定。

## Future API Alignment

- 未来接入真实接口时，优先保持 `CareClockInDraft` 与 `ServiceClockInRecord` 的语义稳定，只替换数据来源。
- 接口设计需要覆盖最小闭环字段：任务标识、服务对象、班次、打卡方式、到场时间、位置、异常说明、确认人、确认时间。
- admin 与 nani 不应直接共享前端模型定义；应由后端契约承担跨端一致性，再在两端分别做映射。
- 若真实接口无法一次性支持主管确认闭环，应优先保证打卡创建与状态查询可用，再分阶段补确认动作。

## Rollout Strategy

- 第一阶段: 先在前端 mock 层增加新页面与派生读模型，不动真实接口。
- 第二阶段: 若后续接后端，再把 `CareClockInDraft` / `ServiceClockInRecord` 对齐到真实契约。
- 当前没有 feature flag，代码回滚即为唯一回退手段。

## Residual Risks

- 当前 mock 协同是单端内存或 localStorage 语义，不代表多端实时同步效果。
- 若未来接入真实扫码、NFC 或定位能力，字段校验与错误态需要重新定义。
- 主管确认目前仍是前端演示动作，不能等价于正式审计签核。