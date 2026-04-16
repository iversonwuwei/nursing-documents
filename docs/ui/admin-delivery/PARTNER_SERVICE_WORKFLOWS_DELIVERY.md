# Partner and Assessment Configuration Workflows Delivery Unit

## Scope

- Entry routes:
  - `src/app/organizations/partners/page.tsx`
  - `src/app/organizations/partners/new/page.tsx`
  - `src/app/nursing/[module]/page.tsx`
- Affected users: 规则配置专员、评估主管、机构运营、协同机构管理人员
- Rollout stage: 定点机构协同页 Fluent 收口与帮助承接批次

## User Impact

- 定点机构继续区分评估机构和护理服务机构，但协同页首屏会先展示场景化总览和对象池，再把协同边界与帮助入口后置。
- `/organizations/partners/new` 主区只保留定点机构录入闭环和主数据表单，机构边界与页面帮助后置到信息轨。
- `/nursing/packages` 不再作为服务包配置，而是承接护理项库与评定规则集。
- `/nursing/plans` 不再作为服务计划，而是承接认定方案模板。
- 页面配置的输出会直接被 `/elderly/checkin` 和 `/financial` 消费。
- `/organizations/partners` 需要显式帮助页承接评估机构/护理服务机构的角色边界，不再把整套说明塞在对象卡片后面。

## Workflow Design

### 1. 协同评估机构治理

- 基本链路：录入机构 -> 指定机构类型 -> 启用 -> 进入首评、复评或抽检协同
- 关键要求：评估机构是认定协同对象，护理服务机构仅作为下游协作背景保留，不再主导当前方案

### 2. 护理项库治理

- 基本链路：创建护理项草稿 -> 启用护理项 -> 被规则集和模板引用
- 关键字段：名称、分类、场景、适用等级、时长、证据要求、说明、状态

### 3. 评定规则集治理

- 基本链路：创建规则集草稿 -> 提交复核 -> 发布生效 -> 个案消费 -> 停用旧版本
- 关键字段：版本、场景、阈值口径、评分区间、关联护理项、质控门禁

### 4. 认定模板治理

- 基本链路：创建模板草稿 -> 提交复核 -> 启用模板 -> 个案自动匹配 -> 归档历史模板
- 关键字段：场景、目标等级、关联规则集、关联护理项、结论摘要、后续动作

## Data Source

- Institution source: `master-data-workflow`
- Assessment configuration source: `assessment-config-workflow`
- Consumer routes: `checkin/page.tsx` and `financial/page.tsx`

## Health Signals

- Healthy signal: 护理项、规则集和模板能独立治理，并能被个案页命中。
- Healthy signal: 评估机构启用后能进入协同视图，不与护理服务机构选择冲突。
- Healthy signal: `/organizations/partners` 在 institutional/home 场景下都能先呈现场景化总览，再展开对象协同细节。
- Failure signal: `/nursing/packages` 仍显示服务套餐心智，`/nursing/plans` 仍显示服务计划心智，或协同页重新堆回长说明首屏。

## Verification

- Minimum gate: `npm run lint`
- Stronger gate: `npm run lint && npm run build`
- Documentation gate: `npm run docs:build`
- Manual paths:
  - 在 `/nursing/packages` 创建护理项和规则集
  - 在 `/nursing/plans` 创建认定模板
  - 在 `/elderly/checkin` 查看个案命中的规则集和模板
  - 在 `/organizations/partners` 的 institutional/home 两个场景下检查对象池、展开详情和帮助入口

## Rollback

- Revert this note together with `assessment-config-workflow.ts` and `NursingWorkflowPages.tsx` changes.
- If regressions appear, fallback is the previous package/plan demo pages, but会恢复错误的服务化主叙事。
