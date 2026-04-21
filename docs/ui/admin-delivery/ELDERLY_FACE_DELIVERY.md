# Elderly Face Delivery Unit

## Scope

- Entry route: src/app/elderly/face/page.tsx
- Related entry points: src/app/elderly/page.tsx, src/app/elderly/[id]/page.tsx
- Affected users: 前台接待、入住接待、护理主管、门禁或设备协同人员
- Rollout stage: 第二十一批人脸录入从前端 mock 切换到 Elder Service 持久化链路

## Change Summary

- `/elderly/face` 页面不再读取 `src/lib/mock/face-enrollment-workflow.ts` 和 `admission-workflow`，改为读取真实 API 队列并写入 Elder Service。
- 新增人脸录入 API 闭环：开始采集、采集单角度、确认激活、退回重录；状态口径仍保持 `待录入/采集中/待确认/已生效/需重录`。
- 数据持久化到 Elder Service（Elder profile face 字段），刷新页面后状态、质量分和备注可回放，不再依赖浏览器 localStorage。
- 老人列表和详情页入口参数语义保持不变（`selected`、`entry`、`scene`），仅替换数据来源。

## Data Source

- Route type: client workflow page
- Primary source: `src/lib/services/admin-face-services.ts`
- Transport chain: 前端 `/api/elders/*` 代理 -> Admin BFF `/api/admin/elders/*` -> Elder Service `/api/elders/*`
- Backend contract: `ElderContracts.cs` 新增 face enrollment request/response DTO
- Persistence: Elder Service `ElderProfileEntity` 新增人脸录入字段（状态、角度、质量、操作人、终端、备注、时间）

## UI States

- Loading state: 首次加载队列显示加载态，动作按钮在提交中禁用避免重复写入。
- Empty state: 当前筛选无对象时显示显式空态并保留返回老人列表入口。
- Error state: 后端失败、字段校验失败、激活前置条件不足时统一显示错误提示并阻断写入。
- Mobile impact: 主区继续优先展示队列与当前对象卡，右轨后置保持不变。

## Workflow And Health Signals

- Healthy signal: 队列状态与对象卡状态来自同一后端记录，刷新后仍一致。
- Healthy signal: 开始采集 -> 三角度采集 -> 激活/重录可在单页闭环并持久化。
- Failure signal: 激活后状态未变、重录后角度未清空、或不同入口定位到同一老人时状态不一致。
- Stable selectors: 录入队列、角度采集按钮、激活按钮、重录按钮保持稳定测试标识。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Backend gate: Elder Service / Admin BFF / BuildingBlocks `dotnet build`
- Docs gate: npm run docs:build
- Manual path: 老人列表或详情进入人脸录入 -> 选择对象 -> 采集三角度 -> 确认激活 -> 刷新后状态保持
- Regression focus: 已生效对象查看、重录分支、入口参数定位、空搜索结果

## Rollback

- Revert this note together with `/elderly/face` and related elderly entry-link changes.
- Fallback is the previous local mock workflow page and localStorage state path.

## Dependencies And Risks

- New dependency or script: none
- Known risks: 当前仍是“状态流 + 质量摘要”口径，不包含真实摄像头上传与门禁设备联调。
- Unverified items: 真摄像头权限、图片文件上传、设备 SDK 兼容性不在本次交付范围。
