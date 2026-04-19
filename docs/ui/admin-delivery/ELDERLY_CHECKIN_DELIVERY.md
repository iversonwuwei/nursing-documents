# Elderly Checkin Delivery Unit

## Scope

- Entry route: `src/app/elderly/checkin/page.tsx`
- Affected users: 评估员、评估主管、复评协同机构联系人
- Rollout stage: 个案评定主流程 live 化第一阶段

## User Impact

- `/elderly/checkin` 现在正式承接个案评定中心，而不是“办理入住”。
- 页面围绕申请受理、AI 辅助建议、人工认定和认定生效组织主流程。
- 规则匹配、模板匹配和协同机构在当前阶段若未接通真实服务，必须显式展示 `Pending Integration`，而不是继续展示本地 store 数据。
- 页面输出语义改为“认定结论与服务建议”，不再把“服务计划生成”作为主叙事。

## Data Source

- Primary sources:
  - `/api/assessments` -> Admin BFF -> Elder Service assessment persistence
  - `/api/admin/ai/admission-assessment` -> AI Orchestration
- Matching logic:
  - 个案根据当前状态映射为待人工确认、计划已生成、已入住
  - AI 建议在创建个案时由 Admin BFF 先调用 admission-assessment，再把结果持久化到 assessment case

## UI States

- Loading state: 初次进入页面时加载真实 assessment case 列表。
- Empty state: 若无评估申请，应保留新建申请入口并提示先从老人新增或资料导入进入。
- Error state: assessment API 不可用、AI 创建失败、非法状态流转时应显式暴露；未接通模块显示 `Pending Integration`。
- Mobile impact: 长页面中的认定依据、协同机构和结论预览卡片需要验证窄屏可读性。

## Health Signals

- Healthy signal: 个案页能够稳定显示真实 assessment case、AI 建议和人工认定结果。
- Healthy signal: 认定状态在列表、详情卡和生效动作上保持一致。
- Failure signal: live 模式仍出现 `assessment-workflow` / `assessment-config-workflow` / `master-data-workflow` 的本地数据痕迹。

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`，以及后端 `dotnet build nursing-backend-services.slnx`
- Manual path:
  - 在 `/elderly/checkin` 创建一条个案评定申请
  - 确认创建后列表出现真实 assessment case，并显示 AI 建议
  - 完成人工认定后，页面显示真实认定结论
  - 对 `计划已生成` 个案执行生效，状态推进到 `已入住`

## Rollback

- Revert this note together with assessment API contracts, Elder/Admin BFF assessment routes, and `checkin/page.tsx` live implementation.
- If regressions appear, fallback is the previous shared mock flow wording, but会恢复错误的数据来源与产品定位。
