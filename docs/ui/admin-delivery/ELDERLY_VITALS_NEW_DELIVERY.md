# Elderly Vitals New Delivery Unit

## Scope

- Entry route: src/app/elderly/vitals/new/page.tsx
- Affected users: 护士、护理主管、巡诊协同人员
- Rollout stage: 第十六批剩余新建能力补齐

## User Impact

- “批量录入”按钮改为真实生命体征录入入口。
- 新录入记录提交后直接回流体征列表，并进入当日录入口径。
- 首批先做单条可追踪录入，后续再扩展真批量模式。

## Data Source

- Route type: client form page
- Primary sink: care service workflow shared store
- Downstream link: `/elderly/vitals?selected=...&entry=elderly-vitals-new`

## UI States

- Loading state: 提交按钮展示保存中。
- Empty state: 默认空表单。
- Error state: 老人、房间、血压、心率、体温、血氧、血糖、记录人缺失时显式提示。
- Mobile impact: 生命体征字段较密集，需保证窄屏下仍可逐项录入。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Manual path: 录入生命体征 -> 列表回流 -> 今日录入口径更新

## Rollback

- Revert this note together with `/elderly/vitals/new` and care service workflow vitals changes.
- Fallback is to restore the previous static vitals list-only behavior.