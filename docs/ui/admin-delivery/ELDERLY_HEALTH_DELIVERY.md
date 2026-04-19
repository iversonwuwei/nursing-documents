# Elderly Health Delivery Unit

## Scope

- Entry route: src/app/elderly/health/page.tsx
- Affected users: 护理主管、巡诊护士、用药协同用户
- Rollout stage: 健康档案页主区收口与帮助后置批次

## User Impact

- 长者健康档案页改为直接读取 Health Service 真实健康摘要，而不是本地 HEALTH_RECORDS 与硬编码用药数据。
- 主区优先保留档案列表、指标汇总和风险摘要；右轨只展示基于实时档案生成的辅助建议，不再依赖本地示例对象。
- 保持健康档案页作为长者健康总览入口，不改变详情入口，但空态与错误态都要显式区分真实接口不可用。

## Data Source

- Route type: client archive page with live health archive query and local search state
- Primary sources: Admin BFF `/api/admin/health/archives`，由 Health Service 健康摘要与 Elder Service 主档合并返回
- Downstream links: elderly detail route links derived from live elderId
- Dependent systems: Admin Next route proxy, Admin BFF, Health Service, Elder Service, 本地 PostgreSQL seed 数据

## UI States

- Loading state: 首屏和刷新动作必须显式展示 live health archive 加载反馈。
- Empty state: 搜索后无匹配档案或健康库尚未写入样本时，应显式提示无结果，而不是只保留空表格或空卡片。
- Error state: Health Service / Admin BFF 返回失败时显式暴露错误，不再继续渲染本地健康样本成功态。
- Mobile impact: 五列统计卡、双栏 AI 区和多列表格在窄屏下需要验证横向滚动与阅读顺序。
- Help state: 通过后置信息轨查看巡诊边界和帮助入口，不再把长说明重新堆回主区。

## Health Signals

- Healthy signal: 健康档案列表、统计卡和右轨辅助建议都基于同一批 live health archive 返回，且 elderId 可稳定跳转到详情页。
- Failure signal: 搜索结果、统计卡与档案表不一致，或页面仍出现硬编码用药对象与本地健康样本。
- Verification proxy: lint 通过；行为改动时加 build 与健康档案流人工回归。

## Verification

- Minimum gate: npm run lint
- Stronger gate for behavior changes: npm run lint and npm run build
- Manual path: 验证搜索、健康指标表、AI 巡诊建议和用药提醒可共同表达当前档案状态

## Rollback

- Revert this delivery note and any future elderly health route changes together.
- If regressions appear, fallback is the previous local health archive composition，并移除 `/api/admin/health/archives` 读取链路。
