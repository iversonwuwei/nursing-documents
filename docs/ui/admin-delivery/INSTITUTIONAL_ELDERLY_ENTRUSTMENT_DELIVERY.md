# Institutional Elderly Entrustment Delivery Unit

## Scope

- Entry routes:
  - `src/app/elderly/new/page.tsx`
  - `src/app/elderly/[id]/edit/page.tsx`
  - `src/app/elderly/checkin/page.tsx`
  - `src/app/elderly/entrustment/page.tsx`
- Affected users: 入住接待、档案专员、护理主管、机构运营
- Rollout stage: 机构养老委托页主区收口与帮助后置批次

## User Impact

- 机构入住建档新增委托来源、月补贴和固定服务项目。
- 老人编辑页改为真实可编辑 workflow 页，不再是静态占位页。
- 机构个案评定页可见委托与补贴摘要，认定不再脱离承接上下文。
- 新增“机构委托入住工作台”，集中查看政府委托、企业委托、待补录和服务中对象。
- 编辑页与委托工作台主区优先保留表单、对象池和执行动作，边界说明与帮助入口后置到信息轨。

## Data Source

- Primary source: Elder Service 的 elder profile 主档
- Compatibility source: `assessment-workflow` 共享 store，仅用于认定中心与委托工作台兼容快照
- Derived view source: 机构工作台按字段完整性 + `AdmissionStatus` 推导阶段
- Rollback path: 回退 elder profile 写链路、新增字段和工作台路由，恢复现有入住建档 / 认定链路

## UI States

- Loading state: 新建提交和编辑保存按钮显示提交中
- Empty state: 委托工作台为空时保留建档和认定入口
- Error state: 委托类型、补贴金额或服务项目缺失时显式报错
- Mobile impact: 新字段卡片和工作台动作保持单列可达
- Help state: 通过后置信息轨查看委托边界和帮助入口，不再把长说明塞回主区。

## Health Signals

- Healthy signal: 建档后对象立即出现在机构委托工作台和个案评定页
- Healthy signal: 编辑补齐字段后，对象从待补录阶段迁移到正确阶段
- Healthy signal: 长者详情页、编辑页、委托工作台看到相同的委托类型、补贴和服务项
- Healthy signal: 编辑页主区、委托工作台对象池、右轨摘要与帮助入口围绕同一委托对象口径保持一致
- Failure signal: 编辑页仍为静态假页，或工作台、认定页、详情页看到的委托数据不一致

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`
- Docs gate: `npm run docs:build`
- Manual path:
  - 新建一位机构委托老人并跳转认定页
  - 打开对应 `/elderly/[id]` 确认委托字段从真实主档回显
  - 打开委托工作台确认阶段、补贴和服务项
  - 编辑已有老人并确认工作台同步刷新

## Rollback

- 回退本交付记录和对应四个入口页改动
- 若新工作台回归失败，可先下线 `/elderly/entrustment` 与入口按钮，保留表单字段扩展
