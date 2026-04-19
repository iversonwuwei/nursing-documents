# Financial Delivery Unit

## Scope

- Entry route: `src/app/financial/page.tsx`
- Affected users: 财务结算人员、评估主管、质控复核人员
- Rollout stage: 评定机构模型纠偏后的结算与质控页 live read-only 收口

## User Impact

- 财务页不再展示护理服务结算，而是承接评定服务结算与质控工作台。
- 页面主视图只保留真实 Billing 摘要、账单队列、通知状态和票据风险，不再回退到前端评定结算单。
- 页面在链路异常时保留显式 live 错误态与空态，而不是静默切回本地 demo workflow。
- 页面当前阶段是 live read-only；若后续恢复真实开票动作，应以 Billing 后端能力为前提，而不是重新挂回前端 assessment store。

## Data Source

- Primary source: `src/lib/services/admin-module-services.ts` 读取的 Billing 摘要与账单队列
- Supporting source: `src/lib/ai/admin-ai-api.ts` 提供只读稽核入口链接，不参与账单事实源
- Contract note: 前端不再以 `assessment-workflow` 或 `assessment-config-workflow` 作为财务页事实源

## UI States

- Loading state: 首屏等待 Billing 摘要与账单队列返回时，页面显示明确同步态。
- Empty state: 若真实 Billing 暂无账单，页面需展示 live 空态，不混入本地结算单。
- Error state: Billing 或通知链路不可用时，页面显示明确错误态和恢复指引，不切回 demo 结算视图。
- Mobile impact: KPI、结算单列表和明细需验证窄屏堆叠和点击可达性。

## Health Signals

- Healthy signal: 结算页的 KPI、列表、详情与右侧状态说明全部来自真实 Billing 摘要或账单。
- Healthy signal: 页面在 live 链路失败时停留在显式错误态，不出现 `Demo Fallback` 或 assessment 结算单。
- Failure signal: 页面重新混入本地结算 store、assessment 派生账单或 demo 开票按钮。

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`
- Documentation gate: `npm run docs:build`
- Manual path:
  - 进入 `/financial`
  - 确认标题、KPI、账单列表和详情都来自真实 Billing 数据
  - 人为制造接口失败时，确认页面显示 live 错误态而不是 demo 回退
  - 确认首屏不再出现评定结算单、资料门禁按钮或 demo 开票入口

## Rollback

- Revert this note together with `financial/page.tsx` changes.
- If regressions appear, rollback is the previous mixed live-plus-demo financial page, but that will restore dual source-of-truth risk.
