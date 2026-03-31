# Elderly New Delivery Unit

## Scope

- Entry route: src/app/elderly/new/page.tsx
- Affected users: 档案录入、入住接待、护理主管用户
- Rollout stage: 第十一批长者工作流子路由治理说明

## User Impact

- 新增长者页当前承接基础档案、健康信息和紧急联系人录入，并在保存后返回长者列表。
- 当前交付单元先固定表单入口职责、验证门禁和回滚路径，不修改提交流程或字段结构。
- 保持新增表单为单页录入流程，不引入分步向导或额外校验逻辑。

## Data Source

- Route type: client form page
- Primary source: local form state with simulated submit delay
- Downstream dependency: next/navigation router push to elderly list

## UI States

- Loading state: 提交时通过 loading 按钮反馈保存中。
- Empty state: 当前依赖表单默认空值和原生 required 校验；后续若接真实入库接口需补字段级空态提示。
- Error state: 当前保留 form-level error 占位，后续接真实提交失败时应显式展示。
- Mobile impact: 三段表单卡片和底部操作区需要验证窄屏下的滚动与提交可达性。

## Health Signals

- Healthy signal: 新增页可完整录入基础资料并稳定提交返回长者列表。
- Failure signal: 提交无反馈、返回路径错误，或表单分组与字段职责漂移。
- Verification proxy: lint 通过；行为改动时加 build 与新增长者流程人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证填写必填项后可提交并返回 `/elderly`

## Rollback

- Revert this delivery note and any future elderly new route changes together.
- If regressions appear, fallback is the current single-page local form submit flow.