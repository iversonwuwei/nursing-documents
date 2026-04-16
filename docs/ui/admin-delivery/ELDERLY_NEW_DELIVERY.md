# Elderly New Delivery Unit

## Scope

- Entry route: src/app/elderly/new/page.tsx
- Affected users: 档案录入、入住接待、护理主管用户
- Rollout stage: 长者新建页主区收口与帮助后置批次

## User Impact

- 新增长者页现在承接“首批可执行主数据 + 评估输入”录入，而不是孤立档案表单。
- 提交后不再直接返回长者列表，而是进入入住审核页并自动选中新建记录。
- 页面明确提示当前交付范围：录入、AI 建议、人工确认和入册闭环的第一步。
- 主区只保留新建闭环说明与录入表单，页面定位、人工边界和完整帮助迁移到信息轨与帮助页入口。

## Data Source

- Route type: client form page
- Primary source: shared admission-workflow form model and校验规则
- Downstream dependency: addAdmissionApplication 写入 shared store，并跳转 `/elderly/checkin?selected=...`

## UI States

- Loading state: 提交时通过 loading 按钮反馈保存中。
- Empty state: 当前依赖表单默认空值和统一校验函数；后续接真实接口时需补字段级提示。
- Error state: 必填项缺失、年龄或 ADL 非法、联系电话不完整、特护申请无风险备注时，显式展示 form-level 错误。
- Mobile impact: 三段表单卡片和底部操作区需要验证窄屏下的滚动与提交可达性。
- Help state: 通过右侧帮助入口承接页面定位和认定闭环说明，不再把培训性文字堆回主区。

## Health Signals

- Healthy signal: 新增页可完整录入最小字段集并稳定跳转到带 `selected` 参数的入住审核页，且主区、信息轨与帮助入口保持同一闭环口径。
- Failure signal: 提交无反馈、对象 ID 丢失、返回路径错误，或表单与审核页校验口径不一致。
- Verification proxy: lint 通过；行为改动时加 build 与新增长者流程人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证填写必填项后可提交，并跳转到 `/elderly/checkin?selected=...` 且自动定位对象

## Rollback

- Revert this delivery note and any future elderly new route changes together.
- If regressions appear, fallback is the旧的单页本地表单并直接返回 `/elderly` 的流程。
