# Financial Delivery Unit

## Scope

- Entry route: `src/app/financial/page.tsx`
- Affected users: 财务结算人员、评估主管、质控复核人员
- Rollout stage: 评定机构模型纠偏后的结算与质控页升级

## User Impact

- 财务页不再展示护理服务结算，而是承接评定服务结算与质控工作台。
- 页面围绕评定案件、资料完整性、人工调整、质控风险和评估费结算进度组织信息。
- 页面可见个案对应的规则集和模板上下文，避免结算与认定脱节。

## Data Source

- Primary source: `assessment-workflow` 共享个案 store 派生出的结算单
- Supporting source: `assessment-config-workflow` 提供规则集和模板上下文
- Contract note: 待认定确认的个案不会提前进入结算视图

## UI States

- Loading state: 当前为本地 mock，后续接真实结算接口时补批次加载与提交反馈。
- Empty state: 若没有已进入结算阶段的案件，页面需提示先完成个案认定。
- Error state: 资料缺失、人工调整无依据、规则或模板缺失时，应在页面上可见。
- Mobile impact: KPI、结算单列表和明细需验证窄屏堆叠和点击可达性。

## Health Signals

- Healthy signal: 结算页只展示已进入认定闭环的案件。
- Healthy signal: 结算页可见规则版本、模板和资料完整性状态。
- Failure signal: 仍以“服务计划”“医保申报”“基金承担”为页面主话术，或结算单无法关联认定上下文。

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`
- Documentation gate: `npm run docs:build`
- Manual path:
  - 进入 `/financial`
  - 确认标题、KPI 和列表语义已经切到评定服务结算
  - 选中一条结算单后可见规则集或模板信息

## Rollback

- Revert this note together with `financial/page.tsx` changes.
- If regressions appear, fallback is the previous demo finance wording, but会恢复与评定机构模型不一致的表达。
