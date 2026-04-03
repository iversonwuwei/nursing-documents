# Elderly Checkin Delivery Unit

## Scope

- Entry route: `src/app/elderly/checkin/page.tsx`
- Affected users: 评估员、评估主管、复评协同机构联系人
- Rollout stage: 评定机构模型纠偏后的核心个案评定页升级

## User Impact

- `/elderly/checkin` 现在正式承接个案评定中心，而不是“办理入住”。
- 页面围绕申请受理、AI 辅助建议、人工认定、规则匹配、模板匹配、协同机构和认定输出组织信息。
- 页面会展示当前个案匹配到的规则集、认定模板和护理项，避免认定依据隐形化。
- 页面输出语义改为“认定结论与服务建议”，不再把“服务计划生成”作为主叙事。

## Data Source

- Primary sources:
  - `assessment-workflow` 共享个案 store
  - `assessment-config-workflow` 共享配置 store
  - `master-data-workflow` 中的评估机构主数据
- Matching logic:
  - 个案根据当前状态映射为首次认定、复评复核或抽检回访场景
  - 根据确认等级或 AI 建议等级匹配有效规则集和启用模板

## UI States

- Loading state: 当前以本地 store 同步为主，后续接真实评定服务时补阶段反馈。
- Empty state: 若无评估申请，应保留新建申请入口并提示先从老人新增或资料导入进入。
- Error state: 配置缺失、模板未命中、人工调整未写说明、协同机构未配置时应显式暴露。
- Mobile impact: 长页面中的认定依据、协同机构和结论预览卡片需要验证窄屏可读性。

## Health Signals

- Healthy signal: 个案页能够稳定显示匹配到的规则集、模板和护理项。
- Healthy signal: 认定状态、协同机构状态和结算前置状态口径一致。
- Failure signal: 页面仍出现“入住审核”“服务计划生成”等旧语义，或个案无法匹配配置依据。

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`
- Manual path:
  - 从 `/elderly/new` 或 `/elderly/import` 进入本页
  - 选中个案后可见规则集、模板和护理项
  - 完成人工认定后，右侧显示认定结论与服务建议预览

## Rollback

- Revert this note together with `checkin/page.tsx` changes.
- If regressions appear, fallback is the previous shared mock flow wording, but会恢复错误的产品定位。
