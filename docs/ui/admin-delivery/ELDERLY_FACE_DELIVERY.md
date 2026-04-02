# Elderly Face Delivery Unit

## Scope

- Entry route: src/app/elderly/face/page.tsx
- Related entry points: src/app/elderly/page.tsx, src/app/elderly/[id]/page.tsx
- Affected users: 前台接待、入住接待、护理主管、门禁或设备协同人员
- Rollout stage: 第十八批老人管理流程补齐

## User Impact

- 人脸录入从静态占位页升级为真实闭环：选择老人 -> 采集校验 -> 人工确认激活。
- 老人列表与老人详情页需要提供稳定入口，避免操作人员在导航中二次检索。
- 老人列表需要同步展示当前人脸状态与快捷动作，让“谁还没录、谁需重录、谁已生效”在主台账里可见。
- 首批仍使用 mock 采集与质检状态，但页面必须能表达待录入、采集中、待确认、已生效和需重录等治理状态。
- 保持流程简洁：不拆新路由，不做复杂设备调试页，首批在单页内完成选择、采集和确认。

## Data Source

- Route type: client workflow page
- Primary sources: elderly live registry + face enrollment workflow shared store
- Related dependencies: 入住闭环生成的 live elderly list 作为可录入对象来源
- Downstream links: 老人列表与详情页跳转 `/elderly/face?selected=<elderlyId>&entry=...`

## UI States

- Loading state: 采集、质检确认和激活动作需展示进行中状态，避免重复点击。
- Empty state: 当前无匹配老人或无待处理录入任务时，应显式提示并保留入口。
- Error state: 未选择老人、采集样本不足、质量校验未通过或备注缺失时，必须阻止激活并给出明确原因。
- Mobile impact: 单页流程在窄屏下应优先展示当前选中老人和采集步骤，列表允许纵向堆叠。

## Workflow And Health Signals

- Healthy signal: 录入页能稳定展示 live elderly list 中的可录入对象，并把录入状态回写到同一 shared store 中。
- Healthy signal: 选中老人后，操作人员能在单页内完成采集、确认和激活，不依赖额外隐藏入口。
- Failure signal: 页面状态与列表状态不一致、激活后仍停留在待确认、或详情与列表无法定位同一录入对象。
- Stable selectors: 录入队列、采集步骤、质检摘要、激活动作和最近完成记录需要保留稳定可测选择器。

## Verification

- Minimum gate: npm run lint
- Stronger gate: npm run lint and npm run build
- Automated path: Playwright smoke 通过老人详情入口进入 `/elderly/face` 并完成激活主链路
- Manual path: 老人列表或详情进入人脸录入 -> 选择待录入老人 -> 完成采集步骤 -> 激活成功 -> 列表状态更新
- Regression focus: 已生效记录查看、采集中继续处理、待确认激活动作和空搜索结果

## Rollback

- Revert this note together with `/elderly/face` and related elderly entry-link changes.
- Fallback is the previous static face registry placeholder page with no active enrollment workflow.

## Dependencies And Risks

- New dependency or script: none
- Known risks: face workflow page will still be mock-driven in this delivery unit, so device or camera integration remains a later phase.
- Unverified items: 真摄像头权限、设备侧照片上传和门禁联调不在本次交付范围。